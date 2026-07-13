# React Native Expo Template

A template for building React Native apps with [Expo](https://expo.dev), organized around a feature-based architecture.

## Project Structure

The app follows a **feature-first, layered** structure under `src/`. Routes stay
thin and delegate to feature screens; shared UI lives in `common/`; technical
infrastructure lives in `core/`.

```text
src/
├── app/                        # Expo Router routes — thin wrappers only
│   ├── _layout.tsx             # Root layout: ThemeProvider + splash + NativeTabs
│   ├── index.tsx               # "/"        → re-exports Home screen
│   └── explore.tsx             # "/explore" → re-exports Explore screen
│
├── common/                     # Shared, cross-feature code (UI-oriented)
│   ├── components/             # Reusable primitives & shared widgets
│   │   ├── themed-text.tsx     #   theme-aware Text
│   │   ├── themed-view.tsx     #   theme-aware View
│   │   ├── app-tabs.tsx        #   NativeTabs configuration
│   │   ├── animated-icon.tsx   #   animated splash icon
│   │   ├── external-link.tsx
│   │   └── ui/collapsible.tsx
│   ├── hooks/                  # Shared hooks
│   │   ├── use-color-scheme.ts #   OS light/dark scheme
│   │   └── use-theme.ts        #   resolves scheme → color tokens
│   ├── theme.ts                # Color tokens (light / dark)
│   └── styles.ts               # Shared style helpers
│
├── core/                       # App-wide technical infrastructure
│   └── data/
│       ├── client.ts           # Generic fetch-based HTTP client (apiClient)
│       └── query-keys.ts       # Query-key factory (extend as data grows)
│
└── features/                   # Feature modules (one folder per feature)
    ├── home/
    │   ├── components/         #   hint-row, …
    │   └── screens/            #   home-screen.tsx (+ .styles.ts)
    └── explore/
        └── screens/            #   explore-screen.tsx (+ .styles.ts)
```

### Layer responsibilities

| Layer | Holds | Depends on |
| --- | --- | --- |
| `app/` | Route files + navigation wiring. Re-export feature screens; no business logic. | `features/`, `common/` |
| `features/` | Self-contained feature UI (`screens/`, `components/`). One feature never imports another. | `common/`, `core/` |
| `common/` | Reusable UI primitives, hooks, theme tokens, style helpers. No feature business rules. | — |
| `core/` | App-wide technical infrastructure (HTTP client, query keys). Framework-level, UI-agnostic. | — |

**Dependency direction:** `app → features → common / core`. Shared code moves
*up* the stack (feature → `common`); it never flows sideways between features.

Conventions: `kebab-case` filenames, `PascalCase` components, `useSomething`
hooks, and the `@/` alias for imports rooted at `src/` (e.g.
`@/common/hooks/use-theme`). Screen styles live in co-located `.styles.ts` files
— except under `src/app/**`, where Expo Router would treat them as routes.

For full architecture details, conventions, and guidelines see [docs/architecture/architecture.md](docs/architecture/architecture.md).

For the Claude Code skills and slash commands available in this repo, see [docs/commands.md](docs/commands.md).

## Getting Started

1. Install dependencies

   ```bash
   pnpm install
   ```

2. Start the app

   ```bash
   npx expo start
   ```

3. Open in your preferred environment:
   - [Development build](https://docs.expo.dev/develop/development-builds/introduction/)
   - [Android emulator](https://docs.expo.dev/workflow/android-studio-emulator/)
   - [iOS simulator](https://docs.expo.dev/workflow/ios-simulator/)
   - [Expo Go](https://expo.dev/go)

## Learn More

- [Expo documentation](https://docs.expo.dev/)
- [Expo Router (file-based routing)](https://docs.expo.dev/router/introduction)
