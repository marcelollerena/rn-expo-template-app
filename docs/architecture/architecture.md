# Architecture

This document describes the architecture, conventions, and extension points of the
**rn-expo-template-app** — a React Native + Expo starter organized around a
feature-first, layered structure.

> This is the human-facing source of truth for how the codebase is organized.
> AI-agent guardrails live under [`.claude/`](../../.claude/) (see
> [Commands & Skills](../commands.md)).

## Tech stack

| Area | Choice | Version |
| --- | --- | --- |
| Runtime | Expo SDK | `~57.0.x` |
| Framework | React Native | `0.86.x` |
| UI runtime | React | `19.2.x` |
| Language | TypeScript | `~6.0.x` |
| Routing | `expo-router` (file-based) | `~57.0.x` |
| Navigation UI | `expo-router/unstable-native-tabs` (`NativeTabs`) | — |
| Animation | `react-native-reanimated` + `react-native-worklets` | `4.5.x` / `0.10.x` |
| Gestures | `react-native-gesture-handler` | `~2.32.x` |
| Layout primitives | `react-native-safe-area-context`, `react-native-screens` | — |
| Images | `expo-image` | `~57.0.x` |

All Expo-aligned packages are pinned with a **tilde (`~`)** to the current SDK
major. Do not use `^` or a different SDK major for Expo packages — see
[Dependencies](#dependencies--sdk-pinning).

## Directory structure

```text
src/
├── app/                     # Expo Router routes (thin wrappers only)
│   ├── _layout.tsx          # Root layout: ThemeProvider + splash + NativeTabs
│   ├── index.tsx            # "/" → re-exports the Home screen
│   └── explore.tsx          # "/explore" → re-exports the Explore screen
│
├── common/                  # Shared, cross-feature code
│   ├── components/          # Reusable UI primitives and shared widgets
│   │   ├── themed-text.tsx
│   │   ├── themed-view.tsx
│   │   ├── app-tabs.tsx     # NativeTabs configuration
│   │   ├── animated-icon.tsx
│   │   ├── external-link.tsx
│   │   └── ui/collapsible.tsx
│   ├── hooks/               # Shared hooks (theme + color scheme)
│   │   ├── use-color-scheme.ts
│   │   └── use-theme.ts
│   ├── styles.ts            # Shared style helpers
│   └── theme.ts             # Color tokens (light / dark)
│
├── core/                    # App-wide technical infrastructure
│   └── data/
│       ├── client.ts        # Generic fetch-based HTTP client wrapper
│       └── query-keys.ts    # Query-key factory (placeholder, extend as needed)
│
└── features/                # Feature modules
    ├── home/
    │   ├── components/       # hint-row, …
    │   └── screens/          # home-screen (+ co-located .styles.ts)
    └── explore/
        └── screens/          # explore-screen (+ co-located .styles.ts)
```

## Architectural principles

### 1. Thin route files

Files under `src/app/` define routes and re-export feature screens. Keep them
minimal:

```tsx
// src/app/explore.tsx
export { default } from '@/features/explore/screens/explore-screen';
```

Navigation wiring lives in `src/app/_layout.tsx`, not inside individual features
(unless a feature introduces its own nested navigator).

**Route directories contain only route files.** Expo Router treats every
`.ts`/`.tsx` under `src/app/**` as part of the route tree, so do **not** put
non-route files (co-located `.styles.ts`, helpers, constants) there. For route
and layout files, keep small style objects inline or lift them to `src/common/`.

### 2. Feature-first UI organization

Feature-specific code lives under `src/features/<feature>/`:

- Full screens → `features/*/screens/`
- Feature-only presentational pieces → `features/*/components/`

Avoid importing one feature from another. If code is reused across features, move
it into `src/common/`.

### 3. Shared layer — `common/`

App-wide reusable, UI-oriented pieces: themed primitives (`ThemedText`,
`ThemedView`), shared hooks, theme tokens, and shared style helpers. Do not place
feature business rules here.

### 4. Core layer — `core/`

App-wide technical infrastructure not tied to a feature. Today:

- `core/data/client.ts` — a small `fetch` wrapper (`apiClient.get/post/put/patch/delete`)
  with a `configureApiClient({ baseUrl, headers })` setup hook.
- `core/data/query-keys.ts` — a query-key factory (currently an empty
  placeholder), ready to grow as data fetching is added.

Only add deeper layers (`domain/`, `repositories/`, `services/`) when the app
actually needs them.

## Routing

The app uses Expo Router with route files under `src/app/`.

| Route | File | Screen |
| --- | --- | --- |
| `/` | `src/app/index.tsx` | Home |
| `/explore` | `src/app/explore.tsx` | Explore |

The root layout (`src/app/_layout.tsx`) mounts a `ThemeProvider`, an animated
splash overlay, and the bottom tab bar (`AppTabs`). Tabs are built with
`NativeTabs` from `expo-router/unstable-native-tabs`:

```tsx
<NativeTabs>
  <NativeTabs.Trigger name="index">…Home…</NativeTabs.Trigger>
  <NativeTabs.Trigger name="explore">…Explore…</NativeTabs.Trigger>
</NativeTabs>
```

### React Navigation on SDK 56+

As of Expo SDK 56, **`expo-router` is not compatible with `@react-navigation/*`
imported from app code**. Metro throws a bundle-time error if you import
`@react-navigation/...` directly. Use the `expo-router` subpaths instead:

| Instead of | Import from |
| --- | --- |
| `@react-navigation/native`, `@react-navigation/elements` | `expo-router/react-navigation` |
| `@react-navigation/bottom-tabs` | `expo-router/js-tabs` |
| `@react-navigation/stack` | `expo-router/js-stack` |

```tsx
import { ThemeProvider, DarkTheme, DefaultTheme } from 'expo-router/react-navigation';
```

Do **not** add `@react-navigation/*` as direct dependencies — `expo-router`
provides them.

## Theming

- Color tokens live in `src/common/theme.ts` (`Colors.light` / `Colors.dark`).
- `use-color-scheme.ts` reads the OS scheme; `use-theme.ts` resolves it to the
  active token set (defaulting `unspecified` → `light`).
- Prefer tokens and the themed primitives (`ThemedText`, `ThemedView`) over
  hardcoded colors.

```tsx
import { useTheme } from '@/common/hooks/use-theme';

const colors = useTheme();
```

## Styling

- Keep screen styles in co-located `.styles.ts` files (e.g.
  `home-screen.styles.ts`) — **except** for route/layout files under
  `src/app/**`, where a `.styles.ts` would be picked up as a route. For those,
  keep styles inline or lift them to `src/common/`.
- Avoid large inline style objects in components; move static styles to the
  `.styles.ts` file and keep only truly dynamic values inline.
- Shared visual primitives belong in `common/components/`.

## Icons

Use **`lucide-react-native`** for standard UI iconography. Do not use emoji,
unicode glyphs, `@expo/vector-icons`, or ad-hoc SVGs for standard icons.

```tsx
import { Bell } from 'lucide-react-native';

<Bell size={22} strokeWidth={1.6} />;
```

## Accessibility

Every interactive element (`Pressable`, `TouchableOpacity`, …) must include an
appropriate `accessibilityRole` and a meaningful `accessibilityLabel`.

## Data & state

Three layers, each with one owner:

- **Local screen state** — `useState` / `useReducer`, close to the component.
- **Remote access** — the `apiClient` wrapper in `core/data/`. Feature code
  should consume typed helpers, not raw `fetch`.
- **Cross-screen / persistent client state** — introduce a single store (e.g.
  Zustand) under a shared location when the need arises; do not create parallel
  per-feature stores.

The template ships a minimal data layer on purpose — add a server-cache library
(such as TanStack Query) and a client-state store only when a real feature needs
them, and document them here once they exist.

## Dependencies & SDK pinning

- The project targets **Expo SDK 57**. Expo-aligned packages use `~57.0.x`.
- When adding an Expo package, use `~` to stay on the current SDK line — never
  `^` or a different SDK major.
- To upgrade the SDK, bump `expo` then run `pnpm expo install --fix` so every
  Expo-managed package (including `react`, `react-native`) is realigned.
- Validate any dependency change with `pnpm dlx expo-doctor`, `pnpm exec tsc
  --noEmit`, and `pnpm lint`.

## Conventions

| Thing | Convention |
| --- | --- |
| File names | `kebab-case.ts` / `kebab-case.tsx` |
| React components | `PascalCase` |
| Hooks | `useSomething` |
| Imports across folders | `@/` alias (rooted at `src/`) |
| Route files | Expo Router naming |

## Package manager

This project uses **pnpm**. Common scripts:

```bash
pnpm install          # install dependencies
pnpm start            # start Metro (add -c to clear cache)
pnpm ios              # run on iOS simulator
pnpm android          # run on Android emulator
pnpm lint             # expo lint
```

## Growing the project

1. Start in `features/<feature>/screens/` and `features/<feature>/components/`.
2. Keep `app/` route files as thin wrappers.
3. Move only truly reusable code to `common/`.
4. Move only app-wide infrastructure to `core/`.
5. Document new abstractions here **after** they exist in code, not before.
