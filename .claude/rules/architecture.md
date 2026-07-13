# Architecture

This document describes the current architecture, conventions, and extension points used in this template app.

## Directory Structure

```text
src/
├── app/                       # Expo Router route files
│   ├── _layout.tsx            # Root layout: providers + Stack
│   ├── index.tsx              # Splash route (entry point)
│   ├── onboarding.tsx         # Onboarding carousel route
│   ├── (tabs)/                # Bottom tab navigation group
│   └── design-preview.tsx     # QA-only design system preview route
│
├── common/                    # Shared cross-feature code
│   ├── components/            # Reusable UI primitives and shared widgets
│   ├── helpers/               # Cross-cutting helper utilities
│   ├── hooks/                 # Shared hooks for theme and platform behavior
│   ├── styles.ts              # Shared style helpers
│   └── theme.ts               # Theme tokens and color definitions
│
├── core/                      # App-level technical foundations
│   ├── config/
│   │   └── env.ts             # Lazy-validated EXPO_PUBLIC_* env reader
│   └── data/
│       ├── client.ts          # Generic HTTP client wrapper
│       ├── query-client.ts    # Singleton TanStack QueryClient
│       ├── query-keys.ts      # Shared query key factory
│       └── shopify/           # Shopify Storefront API integration
│           ├── client.ts      # Lazy-init Storefront GraphQL client
│           ├── hooks.ts       # useCollections / useCollection / useProduct
│           ├── queries.ts     # GraphQL operation strings
│           ├── types.ts       # Re-export shim over generated types
│           └── __generated__/ # graphql-codegen output (pnpm codegen)
│
└── features/                  # Feature modules
    ├── home/
    │   ├── components/
    │   └── screens/
    ├── onboarding/
    │   ├── components/
    │   └── screens/
    ├── shop/
    │   ├── cart/              # Cart hooks + checkout sync listener
    │   ├── checkout/          # Checkout Sheet Kit provider
    │   └── store/             # Zustand store (cart id, wishlist, …)
    └── user/
        ├── components/        # Profile member card, order history, preferences
        ├── data/              # Mock user data (local, no API)
        └── screens/           # User profile screen
```

## Target Platforms

This project targets **iOS and Android only**. Web is not a supported platform.

- Do not add web-specific code, platform selects for `web`, or `.web.tsx` file variants.
- Do not add web-only CSS, CSS variables, or browser-specific workarounds.
- If a library or API requires web-specific configuration, skip it.

## Architectural Principles

### 1. Thin Route Files

Files in `src/app/` should stay minimal. Their job is to define routes and re-export feature screens.

Example:

```tsx
export { default } from "@/features/home/screens/home-screen";
```

Keep navigation setup in `src/app/_layout.tsx`, not inside individual features unless a feature introduces its own nested navigator.

**Route directories contain only route files.** Expo Router treats every
`.ts`/`.tsx` file under `src/app/**` as part of the route tree, so do **not**
place non-route files (co-located `.styles.ts`, helpers, constants, etc.) inside
`src/app/` or any group directory. Doing so either adds a junk route/tab (that
crashes when opened) or — when the file is named `_layout.*` (e.g.
`_layout.styles.ts`) — silently drops the **entire group** from the route tree,
which surfaces only in production/EAS builds (the dev server tolerates it). This
is the one exception to the co-located `.styles.ts` convention below: for a route
or layout, keep its small style objects inline in the route file, or lift them to
`src/common/`. Layout-level styles (e.g. tab-bar styling in `(tabs)/_layout.tsx`)
live inline in the layout.

### 2. Feature-First UI Organization

Feature-specific code lives under `src/features/<feature>/`.

- Put full screens in `features/*/screens/`
- Put feature-only presentational pieces in `features/*/components/`
- Avoid importing from one feature into another feature

If code is reused across features, move it into `src/common/`.

**Exceptions — shop infrastructure**: `features/shop/` hosts the canonical
client-state store (`shop-store.ts`) and cart-aware hooks (`use-cart-query`,
`use-cart-mutations`). Other features may import these directly when they
need cart/auth state for presentation (e.g., the home header's cart badge).
Anything beyond cart/auth state should still be lifted to `common/` or
`core/` before crossing a feature boundary.

### 3. Shared Layer in `common/`

Use `src/common/` for app-wide reusable pieces:

- UI primitives such as themed text/view components
- shared hooks
- theme tokens
- cross-platform helpers
- web-only shared styles

This folder should stay framework-oriented and UI-oriented. Do not place feature business rules here.

### 4. Core Layer in `core/`

Use `src/core/` for app-wide technical infrastructure that is not tied to a specific feature.

Current examples:

- `core/config/env.ts`: lazy-validated reader for `EXPO_PUBLIC_*` env vars (throws on first access, not at module load)
- `core/data/client.ts`: generic HTTP client wrapper
- `core/data/query-client.ts`: singleton `QueryClient` mounted by the root layout
- `core/data/query-keys.ts`: shared query key factory
- `core/data/shopify/`: Shopify Storefront GraphQL client, queries, hooks, and codegen output (see "Data and State" below)

As the app grows, this is the right place for:

- API modules
- repository implementations
- query/mutation adapters
- serialization/mapping helpers that are not feature-local

Only add deeper layers such as `domain/`, `repositories/`, or `services/` when the app actually needs them.

## Routing

The app uses Expo Router with route files under `src/app/`.

Current routes:

- `/` -> `src/app/index.tsx` (splash screen, primary entry)
- `/onboarding` -> `src/app/onboarding.tsx` (3-screen onboarding carousel)
- `(tabs)` -> bottom tab group in `src/app/(tabs)/`:
  - `/home` -> `(tabs)/index.tsx` (home screen)
  - `/shop` -> `(tabs)/shop.tsx`
  - `/cart` -> `(tabs)/cart.tsx` (cart is a tab, not a pushed full-screen route — it has no back button)
  - `/loyalty` -> `(tabs)/loyalty.tsx` (Coming soon placeholder)
  - `/favorites` -> `(tabs)/favorites.tsx` ("Heart" tab, Coming soon placeholder)
- `/user` -> `src/app/user.tsx` (user profile, full-screen Stack route)
- `/my-account` -> `src/app/my-account.tsx` (My Account settings, full-screen Stack route pushed from `/user`)
- `/preferences` -> `src/app/preferences.tsx` (Preferences settings, full-screen Stack route pushed from `/user`)
- `/about` -> `src/app/about.tsx` (About / external policy links, full-screen Stack route pushed from `/user`)
- `/design-preview` -> `src/app/design-preview.tsx` (QA-only, not in primary flow)

The root layout is defined in `src/app/_layout.tsx` and currently mounts:

- React Navigation `ThemeProvider`
- headerless `Stack` navigator

If auth flows, modal stacks, or nested route groups are introduced later, document them only once they exist.

## Dependencies and SDK Pinning

This project is on **Expo SDK 57**. All Expo-aligned packages use **tilde (`~`) specifiers** pinned to the current SDK major (e.g. `~57.0.x`).

- When adding a new Expo package, use `~` to stay within the current SDK line. Do not use `^` or pin to a different SDK major.
- Non-Expo packages follow the existing convention in `package.json` (some use `^`, some use `~`).
- If a package requires a different SDK version, raise it as a separate discussion — do not bundle it into a feature PR.

## Accessibility

Every interactive element (`Pressable`, `TouchableOpacity`, etc.) must include:

- `accessibilityRole="button"` (or the appropriate role)
- `accessibilityLabel` with a meaningful description

Follow the pattern established in the home header (`src/features/home/components/header-bar.tsx`).

## Screen Navigation Controls

Every screen that renders its own back/close affordance must use the shared
top-bar primitive — do not roll a new chevron-in-a-Pressable each time.

- **Back button**: use `BackButton` from `src/common/components/back-button.tsx`.
  - Lucide `ChevronLeft` (size 24, strokeWidth 1.6), 40 × 40 round chip, `hitSlop={12}`.
  - `tone="overlay"` for use over imagery (PDP hero), `tone="surface"` over plain cream/paper backgrounds (settings, etc.). Note: the cart is a tab and intentionally has no back button.
  - Defaults: `router.back()` with fallback to `router.replace('/')`. Override via `onPress` only when the screen needs custom navigation (e.g. fallback to a specific route).
- When introducing other top-bar affordances (close X, share, etc.), promote them into shared primitives once a second usage appears. The PDP favorite button is the live exception until a second consumer exists.

## Styling and Theming

### Current Pattern

The current project uses:

- theme tokens from `src/common/theme.ts`
- shared themed components such as `ThemedText` and `ThemedView`
- co-located `.styles.ts` files for screen-level styles

Example:

```tsx
import { styles } from "./home-screen.styles";
```

```tsx
export const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});
```

### Rules

- Keep screen styles in co-located `.styles.ts` files — **except** for route/layout files under `src/app/**`, where a co-located `.styles.ts` would be picked up as a route (see "Thin Route Files"); keep those styles inline or in `src/common/`
- Prefer theme tokens from `common/theme.ts` over hardcoded values
- Avoid large inline style objects in screen components — move static styles to the `.styles.ts` file; only keep truly dynamic values inline
- Shared visual primitives belong in `common/components/`
- `BottomTabInset` is only appropriate for screens rendered inside the tab navigator. Screens pushed as full-screen Stack routes (outside `(tabs)`) should use `useSafeAreaInsets().bottom` instead.

If a future `useThemedStyles` factory is introduced, update this document when it becomes the real project standard.

## Icons

Use **lucide-react-native** for all icons in the app. Do not use unicode characters, emoji, `@expo/vector-icons`, or custom SVG icons for standard UI iconography.

```tsx
import { Bell, ShoppingBag, Heart } from "lucide-react-native";

<Bell size={22} color={Palette.ink} strokeWidth={1.6} />;
```

## Key Rules

Key rules:

- **Use the current repo shape**: route files live in `src/app/`, shared code in `src/common/`, infrastructure in `src/core/`, and feature code in `src/features/`.
- **Keep route files thin**: files in `src/app/` should primarily re-export screens from `src/features/*/screens/` or define top-level layout/navigation wiring.
- **Feature isolation**: feature code lives under `src/features/<feature>/`. Do not import one feature directly into another unless there is a strong reason; move shared pieces to `src/common/`.
- **Shared UI and hooks**: reusable components, theme helpers, and shared hooks belong in `src/common/`.
- **Core infrastructure**: generic API clients, query helpers, and app-wide technical foundations belong in `src/core/`.
- **Styling**: screen styles should live in co-located `.styles.ts` files. Avoid large inline style objects in screen components.
- **Theming**: prefer tokens and helpers from `src/common/theme.ts` and existing themed components such as `ThemedText` and `ThemedView`.
- **Use what exists**: TanStack Query (`core/data/query-client.ts`) is the canonical server-cache layer, and a single Zustand store (`features/shop/store/shop-store.ts`) owns persistent client state. Extend those — do not add parallel data-fetching or state-management mechanisms. Do not introduce deeper layers such as `domain/`, `repositories/`, or `services/` unless the task actually requires them.
- **Imports**: use the `@/` alias for imports from `src/`.
- **Naming**: use kebab-case for filenames, PascalCase for React components, and camelCase for hooks and functions.

When the architecture evolves, update this file so the instructions stay aligned with the codebase.

## Naming Conventions

- Files: `kebab-case.ts` / `kebab-case.tsx`
- React components: `PascalCase`
- Hooks: `useSomething`
- Route files: follow Expo Router naming
- Feature folders: singular or plural is acceptable, but keep each feature internally consistent

## Imports

Use the `@/` alias for app code rooted at `src/`.

Examples:

- `@/common/components/themed-text`
- `@/core/data/client`
- `@/features/home/screens/home-screen`

Prefer alias imports over long relative paths when importing across folders.

## Specification Location

This project stores implementation specifications under `docs/specs/features/`.

- Each spec must live in its own folder under `docs/specs/features/`
- The folder name must be kebab-case and describe the feature or initiative
- The spec file inside that folder must always be named `spec.md`

Example:

```text
docs/specs/features/onboarding-flow/spec.md
```

## Data and State

The app uses three layers, each with one canonical owner:

- **Local screen state** — `useState` / `useReducer`, kept close to the component.
- **Server cache** — `@tanstack/react-query`. The singleton `QueryClient` lives in `src/core/data/query-client.ts` and is mounted in `src/app/_layout.tsx`. Query keys come from the factory in `src/core/data/query-keys.ts`. Feature hooks call `useQuery` / `useMutation` directly; do not hand-roll a fetch+`useState` pattern when the data is server state.
- **Client state that crosses screens or persists** — `zustand`. The single store lives in `src/features/shop/store/shop-store.ts` and uses the `persist` middleware over `AsyncStorage`. Slices today: `cart` (cart id), `wishlist`, `recentlyViewed`, plus a `filters` placeholder. Add new persistent client state as a slice on this store; do not create per-feature stores.

Rules:

- Keep remote access wrappers in `core/data/`. Feature code consumes hooks, not raw `fetch` / GraphQL clients.
- Use selectors (`useShopStore((s) => s.cartId)`) instead of consuming the whole store object.
- The persist `version` field in `shop-store.ts` must be bumped when persisted slice shapes change, paired with a `migrate` callback.

### GraphQL codegen (Shopify Storefront)

The Storefront GraphQL types live in `src/core/data/shopify/__generated__/operations.ts` and are produced by [graphql-codegen](https://the-guild.dev/graphql/codegen) introspecting the live store. Workflow:

1. Edit a query or mutation string in `src/core/data/shopify/queries.ts`.
2. Run `pnpm codegen`. This reads `.env.local` for shop domain, API version, and Storefront public token, then regenerates the operations file.
3. Commit the regenerated file. Consumers import types via the re-export shim in `src/core/data/shopify/types.ts`, never the generated file directly.

The generated folder is excluded from `eslint` (`eslint.config.js`).

**Never hand-edit anything under `__generated__/`.** Treat those files as read-only build artifacts:

- If a generated type is wrong, fix the query in `queries.ts` or the codegen config in `codegen.ts` and re-run `pnpm codegen` — do not patch the output.
- If you need a renamed, derived, or simplified type, add it to the shim in `src/core/data/shopify/types.ts`. The shim is the seam between codegen and consumers.
- Do not import from `./__generated__/operations` outside the shim. The shim should be the only file with that import path.
- Manual changes inside `__generated__/` will be silently overwritten the next time anyone runs `pnpm codegen`, so they will look like phantom regressions in review.

## Guidance For Future Growth

When adding new features:

1. Start with `features/<feature>/screens/` and `features/<feature>/components/`
2. Keep `app/` route files as wrappers
3. Move only truly reusable code to `common/`
4. Move only app-wide infrastructure to `core/`
5. Do not document abstractions before they exist in code
