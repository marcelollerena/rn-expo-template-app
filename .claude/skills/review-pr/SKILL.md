---
name: review-pr
description: Reviews a pull request against this project's React Native / Expo architecture rules ([.claude/CLAUDE.md](.claude/CLAUDE.md), [.claude/rules/architecture.md](.claude/rules/architecture.md)) plus standard quality gates. Use when reviewing a PR, when the user says "review the PR", "/review-pr", or asks for an architecture review of pending changes. Args - optional PR number; optional `worktree` keyword to run inside an isolated git worktree that is cleaned up afterward.
---

Review the target PR against the project's documented architecture and standard React Native / Expo quality gates. Print the full review to the terminal AND post it to the PR automatically when the review completes — no confirmation prompt. The posted review carries a general summary body plus inline file:line comments.

The architecture rules are single-sourced in [.claude/CLAUDE.md](.claude/CLAUDE.md) and project rules under [.claude/rules/](.claude/rules/) (currently [.claude/rules/architecture.md](.claude/rules/architecture.md)). Read every file in `.claude/rules/` before reviewing — do not paraphrase them from this file; cite the rule by linking back to the source.

## 1. Resolve the target PR

- If the args contain a number, treat it as the PR number.
- Otherwise run `gh pr view --json number,headRefName,baseRefName,title,body,files,additions,deletions` to detect the PR for the current branch.
- If no PR is found, abort with a clear message asking the user to either pass a PR number or open a PR first.

## 2. Optional worktree isolation

If the args contain the literal `worktree`:

1. `git fetch origin`
2. `git worktree add ../mobile_app-review-<PR#> origin/<headRef>`
3. Operate inside that worktree for the rest of the review (use absolute paths or `cd` into it).
4. Record the worktree path so step 8 can remove it.

Otherwise, operate in the current checkout.

## 3. Gather diff context

- `gh pr diff <PR#>` — unified diff
- `gh pr view <PR#> --json files` — file-level summary
- `git log origin/<baseRef>..origin/<headRef> --oneline` — commit history

## 4. Architecture review

Check the diff against every rule in [.claude/CLAUDE.md](.claude/CLAUDE.md) and the files under [.claude/rules/](.claude/rules/) (currently [.claude/rules/architecture.md](.claude/rules/architecture.md)). At minimum:

- **Thin route files** — files under `src/app/` re-export feature screens or wire navigation; no business logic. Layout/providers belong in `_layout.tsx`.
- **Feature isolation** — no imports from `src/features/<a>/` into `src/features/<b>/`. If two features need the same code, it belongs in `src/common/`.
- **Layering** — shared UI/hooks/theme live in `src/common/`; app-wide technical infrastructure (HTTP clients, query keys, etc.) lives in `src/core/`.
- **Co-located styles** — screen styles live in sibling `.styles.ts` files. Flag large inline `StyleSheet.create` blocks inside screen components.
- **Theming** — flag hardcoded colors / spacing / typography that should use tokens from `src/common/theme.ts`, `ThemedText`, or `ThemedView`.
- **Imports** — flag long relative paths that cross folders; require the `@/` alias.
- **Naming** — `kebab-case.ts(x)` filenames, `PascalCase` components, `useSomething` hooks, `camelCase` functions.
- **No premature abstractions** — flag new `domain/`, `repositories/`, `services/`, Zustand, or React Query layers unless the diff genuinely needs them.

## 5. React Native / Expo correctness review

- **React hooks rules** — top-level only, no conditional hooks, correct dependency arrays.
- **Lists** — `FlatList` / `SectionList` for variable-length data; `keyExtractor` provided; item renderers and key functions are stable references.
- **Performance** — avoid inline arrow functions and inline object literals passed as props to memoized components or list items; reach for `useMemo` / `useCallback` where it matters.
- **Accessibility** — interactive elements have `accessibilityLabel` and `accessibilityRole`; images have an accessible label or are marked decorative.
- **Platform handling** — `Platform.OS` guards where the code is platform-specific; no web-only or native-only API access without a guard.
- **Reanimated / Worklets** — `'worklet'` directives where required; no JS-thread access from inside a worklet; shared values not mutated during render.
- **Expo Router** — route files stay thin; navigation params typed (typed routes are enabled in this project).
- **TypeScript** — `strict: true` is on. Flag `any`, unjustified non-null assertions, untyped exports, and missing prop types.
- **Security** — no hardcoded secrets, API keys, or tokens; no `eval`; no `dangerouslySetInnerHTML`; sensitive data uses `expo-secure-store`, not `AsyncStorage`.

## 6. Quality gates

Run only scripts that exist in `package.json` (do not invent commands):

- `npm run lint`
- If `typescript` is installed, also run `npx tsc --noEmit`.

Capture output and include pass/fail in the report.

## 7. Output

Print a markdown report with these sections, in order:

- **Summary** — one paragraph, ending with a ship / block recommendation.
- **Architecture findings** — rule violations with file:line and a link back to the rule.
- **Correctness & React Native findings**
- **Quality gate results** — lint + tsc output (pass/fail and key errors).
- **Suggestions** — nice-to-haves, clearly separated from blockers.

Per finding, include: severity (`blocker` / `warning` / `nit`), `file:line`, the rule violated (linked), and a concrete recommendation.

## 8. Post the review to GitHub

After printing the local report (section 7), post it to the PR automatically — do not ask for confirmation.

1. **Derive the review `event`** from the Summary recommendation:
   - any finding with severity `blocker` → `REQUEST_CHANGES`
   - no blockers and no warnings, and lint + tsc both pass → `APPROVE`
   - otherwise → `COMMENT`
2. **Build inline comments.** For each finding that has a concrete `file:line`, only post it inline if that line is part of the PR diff — GitHub rejects inline comments anchored to unchanged lines. Cross-check against `gh pr diff <PR#>` (the diff gathered in step 3). Each inline comment is:
   ```json
   { "path": "<file>", "line": <line>, "side": "RIGHT", "body": "<severity> — <rule link> — <recommendation>" }
   ```
3. **Build the summary body.** Use the Summary paragraph and the ship/block recommendation. Append any findings that have no diff-anchored line (so nothing is dropped), plus the quality-gate results.
4. **Post a single review** via stdin to avoid `-f` quoting problems with multi-line bodies and arrays:
   ```bash
   gh api --method POST repos/:owner/:repo/pulls/<PR#>/reviews --input - <<'JSON'
   { "event": "COMMENT", "body": "<summary body>", "comments": [ /* inline comments */ ] }
   JSON
   ```
   Substitute the real `event`, `body`, and `comments` array.
5. **On failure** (e.g. a `422` because a line is not in the diff), retry once: move the offending findings into the body and post only the valid inline comments. Surface any remaining error to the user.
6. Print the resulting review URL (`html_url` from the response).

## 9. Cleanup (worktree mode only)

This step MUST run even if earlier steps surfaced blockers or threw errors — treat it as a `finally` block:

1. `cd` back to the original repo path.
2. `git worktree remove --force <recorded path>`
3. `git worktree prune`
4. Confirm the path is gone with `git worktree list`. Surface any cleanup error to the user rather than swallowing it.
