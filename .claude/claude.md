# Project Instructions

## Role

You are the main AI coding agent for this project.

Act as a senior engineer. Prioritize simplicity, maintainability, and correctness. Avoid overengineering.

## Application Context

This repository is a **template**. It has no product-specific domain baked in — the code
demonstrates architecture and conventions, not a particular app.

When starting a real app from this template, create a `.claude/docs/` folder and drop
project context there so Claude has more to work with:

- A **PRD** (product requirements document) describing what the app does.
- Any general context: domain overview, target users, business rules, integrations,
  design references, or links to external specs.

Any file placed under `.claude/docs/` is treated as background context for the app under
development. Read it before implementing features so decisions align with the product's
intent. This is separate from `docs/specs/features/` (see "Specification Location"), which
holds implementation specs for individual features.

## Project Principles

- Prefer small, focused changes.
- Do not introduce new abstractions unless there is a clear repeated pattern.
- Follow the existing architecture before proposing a new one.
- Ask before adding new dependencies.
- Prefer readable code over clever code.
- Do not modify unrelated files.

## Workflow

Before implementing:

1. Understand the existing code.
2. Explain the plan briefly.
3. Identify impacted files.
4. Implement the smallest safe change.
5. Run the relevant checks.

After implementing:

1. Summarize what changed.
2. Mention any tradeoffs.
3. Suggest next steps only if necessary.

## Package Manager

Use pnpm by default.

Examples:

- `pnpm install`
- `pnpm add <package>`
- `pnpm lint`
- `pnpm test`

Do not use npm unless explicitly requested.
