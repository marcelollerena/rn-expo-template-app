# Architecture

This document describes the current architecture, conventions, and extension points used in this Expo + React Native template.

## Directory Structure

```text
src/
├── app/                       # Expo Router route files
│   ├── _layout.tsx            # Root app layout and providers
│   ├── index.tsx              # Home route
│   └── explore.tsx            # Explore route
│
├── common/                    # Shared cross-feature code
│   ├── components/            # Reusable UI primitives and shared widgets
│   ├── hooks/                 # Shared hooks for theme and platform behavior
│   ├── global.css             # Shared web styles
│   ├── styles.ts              # Shared style helpers
│   └── theme.ts               # Theme tokens and color definitions
│
├── core/                      # App-level technical foundations
│   └── data/
│       ├── client.ts          # Generic API client wrapper
│       └── query-keys.ts      # Query key factory placeholder
│
└── features/                  # Feature modules
    ├── explore/
    │   └── screens/
    └── home/
        ├── components/
        └── screens/
```

## Architectural Principles

### 1. Thin Route Files

Files in `src/app/` should stay minimal. Their job is to define routes and re-export feature screens.

Example:

```tsx
export { default } from '@/features/home/screens/home-screen';
```

Keep navigation setup in `src/app/_layout.tsx`, not inside individual features unless a feature introduces its own nested navigator.

### 2. Feature-First UI Organization

Feature-specific code lives under `src/features/<feature>/`.

- Put full screens in `features/*/screens/`
- Put feature-only presentational pieces in `features/*/components/`
- Avoid importing from one feature into another feature

If code is reused across features, move it into `src/common/`.

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

- `core/data/client.ts`: generic HTTP client wrapper
- `core/data/query-keys.ts`: place for shared query key factories

As the app grows, this is the right place for:

- API modules
- repository implementations
- query/mutation adapters
- serialization/mapping helpers that are not feature-local

Only add deeper layers such as `domain/`, `repositories/`, or `services/` when the app actually needs them.

## Routing

The app uses Expo Router with route files under `src/app/`.

Current routes:

- `/` -> `src/app/index.tsx`
- `/explore` -> `src/app/explore.tsx`

The root layout is defined in `src/app/_layout.tsx` and currently mounts:

- React Navigation `ThemeProvider`
- splash overlay
- tab navigation via `AppTabs`

If auth flows, modal stacks, or nested route groups are introduced later, document them only once they exist.

## Styling and Theming

### Current Pattern

The current project uses:

- theme tokens from `src/common/theme.ts`
- shared themed components such as `ThemedText` and `ThemedView`
- co-located `.styles.ts` files for screen-level styles

Example:

```tsx
import { styles } from './home-screen.styles';
```

```tsx
export const styles = StyleSheet.create({
  container: {
    flex: 1,
  },
});
```

### Rules

- Keep screen styles in co-located `.styles.ts` files
- Prefer theme tokens from `common/theme.ts` over hardcoded values
- Avoid large inline style objects in screen components
- Shared visual primitives belong in `common/components/`

If a future `useThemedStyles` factory is introduced, update this document when it becomes the real project standard.

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

## Data and State

The current template contains only a lightweight API client and query key placeholder. It does not yet include a committed app-wide state management library such as Zustand or TanStack Query.

Until those are added:

- keep local screen state close to the screen/component
- keep remote access wrappers in `core/data/`
- introduce query/state libraries only when an actual feature needs them

When those libraries are adopted, update this document and the AI instruction files at the same time.

## Guidance For Future Growth

When adding new features:

1. Start with `features/<feature>/screens/` and `features/<feature>/components/`
2. Keep `app/` route files as wrappers
3. Move only truly reusable code to `common/`
4. Move only app-wide infrastructure to `core/`
5. Do not document abstractions before they exist in code
