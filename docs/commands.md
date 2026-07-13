# Commands & Skills

This project ships a set of **Claude Code skills** under
[`.claude/skills/`](../.claude/skills/). Skills are reusable capabilities the AI
agent can apply; some are invoked explicitly as slash **commands**, others are
knowledge references the agent applies automatically when relevant.

> Skill sources and integrity hashes are tracked in
> [`skills-lock.json`](../skills-lock.json). Agent-wide instructions live in
> [`.claude/CLAUDE.md`](../.claude/CLAUDE.md) and [`.claude/rules/`](../.claude/rules/).

## How to use a command

In a Claude Code session, type `/` followed by the skill name, then any
arguments:

```text
/rn-audit project
/review-pr 42
/fix-pr-comments
```

Arguments shown in `[brackets]` are optional.

## Project commands

These are the project's own command-style skills — run them explicitly.

### `/rn-audit [project|git] [focus]`

Read-only React Native / Expo auditor. Runs in a forked, read-only subagent
(never modifies files) and applies Callstack's `react-native-best-practices`.

- `project` (default) — comprehensive audit of the whole project.
- `git` — audit only the current Git working-tree changes (staged, unstaged,
  untracked, renamed, deleted).
- Trailing text is treated as a focus area / path / constraints.

```text
/rn-audit project
/rn-audit git
/rn-audit project startup and list performance
```

Produces a structured report (severity-ranked findings with file:line evidence,
a scorecard, and a readiness assessment). It does **not** change code.

### `/review-pr [pr-number] [worktree]`

Reviews a pull request against this project's architecture rules
([`.claude/CLAUDE.md`](../.claude/CLAUDE.md),
[`.claude/rules/architecture.md`](../.claude/rules/architecture.md)) plus standard
quality gates.

- Optional PR number (defaults to the PR for the current branch).
- Optional `worktree` keyword runs the review inside an isolated git worktree
  that is cleaned up afterward.

```text
/review-pr
/review-pr 42
/review-pr 42 worktree
```

### `/fix-pr-comments [pr-number]`

Walks through a PR's inline and general review comments, asks per-comment
whether to address each one, builds a plan, then applies the changes and pushes.

- Optional PR number (defaults to the PR for the current branch).

```text
/fix-pr-comments
/fix-pr-comments 42
```

## Framework knowledge skills

These are reference skills the agent applies automatically when a task matches;
you can also invoke them explicitly. They do not modify your project by
themselves.

| Skill | Use it for |
| --- | --- |
| `react-native-best-practices` | Performance guidance: FPS, TTI, bundle size, memory, re-renders, animations. |
| `expo-router` | File-based routing, groups, dynamic routes, `NativeTabs`, headers, modals, sheets. |
| `expo-ui` | Native UI with `@expo/ui` (SwiftUI / Jetpack Compose from React). |
| `expo-module` | Authoring native modules/views with the Expo Modules API. |
| `expo-dev-client` | Building and distributing Expo development clients. |

## Adding a new command

1. Create `.claude/skills/<name>/SKILL.md` with YAML frontmatter (`name`,
   `description`, optional `argument-hint`).
2. For an explicit, command-only skill, set `disable-model-invocation: true`.
3. To run it in an isolated subagent, add `context: fork` and restrict tools via
   `allowed-tools` (e.g. `Read, Glob, Grep, Bash` for a read-only tool).
4. Document it in this file.

See [`.claude/skills/rn-audit/SKILL.md`](../.claude/skills/rn-audit/SKILL.md) for a
complete example of a read-only, forked, command-only skill.
