All generated code MUST follow the current architecture defined in `docs/architecture/architecture.md`.

Key rules:

- **Use the current repo shape**: route files live in `src/app/`, shared code in `src/common/`, infrastructure in `src/core/`, and feature code in `src/features/`.
- **Keep route files thin**: files in `src/app/` should primarily re-export screens from `src/features/*/screens/` or define top-level layout/navigation wiring.
- **Feature isolation**: feature code lives under `src/features/<feature>/`. Do not import one feature directly into another unless there is a strong reason; move shared pieces to `src/common/`.
- **Shared UI and hooks**: reusable components, theme helpers, and shared hooks belong in `src/common/`.
- **Core infrastructure**: generic API clients, query helpers, and app-wide technical foundations belong in `src/core/`.
- **Styling**: screen styles should live in co-located `.styles.ts` files. Avoid large inline style objects in screen components.
- **Theming**: prefer tokens and helpers from `src/common/theme.ts` and existing themed components such as `ThemedText` and `ThemedView`.
- **Use what exists**: do not introduce architecture layers such as `domain/`, `repositories/`, Zustand stores, or React Query patterns unless the task actually requires them.
- **Imports**: use the `@/` alias for imports from `src/`.
- **Naming**: use kebab-case for filenames, PascalCase for React components, and camelCase for hooks and functions.

When the architecture evolves, update this file and `docs/architecture/architecture.md` together so the instructions stay aligned with the codebase.
