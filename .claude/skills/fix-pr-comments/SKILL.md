---
name: fix-pr-comments
description: Walks through inline and general review comments on a PR, asks per-comment whether to address them, builds a plan in plan mode, then executes the changes and pushes. Use when the user says "fix pr comments", "/fix-pr-comments", "address review feedback", or asks to iterate on PR review comments. Args - optional PR number (defaults to the PR for the current branch).
---

Iterate on PR review feedback interactively. Architecture rules to honor when applying changes are documented in [.claude/CLAUDE.md](.claude/CLAUDE.md) and the files under [.claude/rules/](.claude/rules/) (currently [.claude/rules/architecture.md](.claude/rules/architecture.md)) — link to them rather than restating them.

## 1. Resolve the target PR

- If the args contain a number, treat it as the PR number.
- Otherwise run `gh pr view --json number,headRefName,baseRefName,author` for the current branch.
- If no PR is found, abort and ask the user to pass a PR number or open one.

## 2. Preconditions

- Working tree must be clean (`git status --porcelain`). If not, abort and ask the user to stash or commit first — never mix unrelated changes into a feedback-fix commit.
- Confirm we are on the PR's head branch. If not, ask the user to switch — do not switch automatically.

## 3. Fetch comments

- Inline review comments: `gh api repos/:owner/:repo/pulls/<PR#>/comments --paginate` — fields of interest: `id`, `path`, `line` (fall back to `original_line`), `body`, `user.login`, `in_reply_to_id`, `pull_request_review_id`.
- General review summary bodies: `gh api repos/:owner/:repo/pulls/<PR#>/reviews --paginate` — keep entries with a non-empty `body`.
- Issue-style top-level comments (the PR conversation comments, the `#issuecomment-...` kind): `gh api repos/:owner/:repo/issues/<PR#>/comments --paginate` — fields `id`, `body`, `user.login`. These have no `path`/`line` and must NOT be skipped; they are a normal source of review feedback and feed the dialogue loop like any other comment.
- Resolve the current user via `gh api user --jq .login` and skip comments authored by them (avoid re-addressing your own past replies). Apply this filter to all three sources above.
- Group reply chains by `in_reply_to_id`: keep the top-level comment plus the latest reply for context; drop intermediate replies. (Only inline review comments have `in_reply_to_id`; issue comments are always standalone.)

## 4. Enter plan mode

Call `EnterPlanMode`. All subsequent edits are gated by it. If already in plan mode, skip.

## 5. Per-comment dialogue loop

For each comment (one at a time, in order):

1. Print the comment with file/line context — read 5 lines of surrounding code via `git show HEAD:<path>` or by reading the file directly. File-less issue comments have no `path`/`line`, so show them without a code snippet.
2. Ask via `AskUserQuestion`:
   - **Apply as-is** — adopt the reviewer's suggestion literally.
   - **Apply with modification** — follow up to capture the user's variation.
   - **Skip** — record a reason to use as the reply on GitHub.
   - **Defer** — leave for a later run; do not include in this run.
3. Record the decision: `{ commentId, file, line, decision, intendedChange, skipReason? }`.

## 6. Consolidate into a plan

Write the plan to the standard plan file. Structure it as:

- A section per file with a checklist of edits to apply.
- Group multiple comments touching the same area into a single edit pass.
- Call out cross-cutting concerns where several comments converge on the same refactor.
- Honor the project's architecture rules when proposing fixes — see [.claude/CLAUDE.md](.claude/CLAUDE.md).

## 7. Exit plan mode and await approval

Call `ExitPlanMode`. Do not edit any source file before the user approves the plan.

## 8. Execute the plan

After approval:

1. Apply edits file-by-file in the order listed in the plan.
2. After each file's edits, re-read the file to confirm the result.
3. Run `npm run lint`, and `npx tsc --noEmit` if TypeScript is installed.
4. If either gate fails, stop and surface the error to the user — do not auto-fix beyond what the comments asked for.

## 9. Commit

- Stage only the files this run touched: `git add <specific paths>`. Never `git add -A` / `git add .`.
- Commit with title `fix: address PR review feedback`. In the body, list the comment IDs / authors addressed — this is the audit trail.

## 10. Push

- Check the upstream: `git rev-parse --abbrev-ref --symbolic-full-name @{u}`.
- If no upstream is configured, ask the user before setting one.
- `git push`. Never `--force` unless the user explicitly asks.

## 11. Optionally reply on GitHub

Ask the user once: should the skill post replies to each addressed comment?

If yes, for every applied (not deferred, not skipped) comment, post a reply — routing by comment type:

- **Inline review comments** → reply on the thread:
  ```
  gh api repos/:owner/:repo/pulls/<PR#>/comments/<id>/replies -f body="Addressed in <commit-sha>"
  ```
- **Issue-style top-level comments** → the replies endpoint does not apply; post a new issue comment instead:
  ```
  gh api repos/:owner/:repo/issues/<PR#>/comments -f body="Addressed in <commit-sha>"
  ```

For `Skip` decisions, post the captured reason instead (via the same routing). For `Defer`, post nothing.

## 12. Final summary

Print a table: `comment` → `decision` → `commit SHA (if applied)` → `reply posted (yes/no)`. Call out deferred items so the user can return to them later.
