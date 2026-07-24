---
name: campwise-project
description: Continue, review, debug, or extend the Campwise ("Ką imam?") camping planner repository. Use for Campwise product features, UI changes, API or database work, authentication, localization, tests, deployment, or architectural questions so changes follow the project's established patterns.
---

# Campwise Project

Use this skill as the fast onboarding guide for Campwise. Treat the repository source as authoritative when it differs from this skill.

## Start every task

1. Read `references/project-map.md`.
2. Inspect `git status --short --branch` and preserve unrelated user changes.
3. Read only the files named by the relevant section of the project map, then follow direct imports as needed.
4. Keep the English and Lithuanian experiences functionally equivalent.

## Implement features

- Preserve the vinext/Next App Router structure and Cloudflare Worker compatibility.
- Extend the existing `PlannerState` and action-based `/api/planner` contract instead of creating parallel state systems.
- Store durable product data in D1. Update both `db/schema.ts` and the runtime `initialize()` statements, then generate a Drizzle migration.
- Derive identity from `getChatGPTUser()`. Never accept ownership or payer identity from a client payload when it can come from the authenticated user.
- Check trip membership before every trip-scoped read or write. Constrain destructive writes by both resource id and owner or trip.
- Validate and bound every client-provided value. Keep the same-origin write check and prepared statements.
- Update realistic guest demo data whenever a new required field or visible capability is added.
- Add all user-facing copy to both locales in `app/PlannerApp.tsx`.

## Validate and hand off

1. Run focused tests for new business logic.
2. Run `npm run db:generate` after schema changes and inspect the SQL.
3. Run `npm run lint`, `npm test`, and fix real failures.
4. Because `.openai/hosting.json` exists, use the Sites build and hosting workflow unless the user explicitly asks for local-only work.
5. Update `references/project-map.md` whenever architecture, persistence, routes, core invariants, or major features change.

Do not commit, push, or open a pull request unless the user asks.
