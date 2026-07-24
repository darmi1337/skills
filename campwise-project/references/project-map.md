# Campwise project map

## Product

Campwise is branded **Ką imam?** in the UI. It coordinates camping trips so a group can:

- keep personal gear inventories;
- create or join trips with six-character invite codes;
- build and assign a shared needs list;
- track each camper's pickup and packing list;
- record trip baskets and calculate equal-share repayments;
- use the complete app in English (`/`) or Lithuanian (`/lt`).

Guest visitors see realistic in-memory demo data. Signed-in users read and write durable D1 data.

## Runtime and deployment

- Next.js App Router UI compiled by vinext and Vite.
- React client application in `app/PlannerApp.tsx`.
- Cloudflare Worker entry point in `worker/index.ts`.
- Cloudflare D1 binding named `DB`.
- Sites configuration in `.openai/hosting.json`; reuse its opaque `project_id`.
- `vite.config.ts` must retain `vinext()`, `sites()`, and the Cloudflare plugin.
- Node.js 22.13 or newer; npm and `package-lock.json`.

## Primary files

| Concern | Source |
| --- | --- |
| English page and authenticated user bootstrap | `app/page.tsx` |
| Lithuanian page | `app/lt/page.tsx` |
| Product UI, demo state, locale copy, client actions | `app/PlannerApp.tsx` |
| Global responsive styling | `app/globals.css` |
| Planner API and runtime schema initialization | `app/api/planner/route.ts` |
| ChatGPT header-based identity | `app/chatgpt-auth.ts` |
| Drizzle schema | `db/schema.ts` |
| Generated migrations | `drizzle/` |
| Expense balancing algorithm | `app/expense-balances.ts` |
| Worker and security headers | `worker/index.ts` |
| Build/deployment adapter | `vite.config.ts`, `build/sites-vite-plugin.ts` |
| Tests | `tests/` |

## State and request flow

`app/page.tsx` or `app/lt/page.tsx` gets the platform-authenticated user and renders `PlannerApp`.

For a signed-in user:

1. `PlannerApp` fetches `GET /api/planner`.
2. The API initializes any missing D1 tables, ensures the profile, and returns one `PlannerState`.
3. Client mutations send `POST /api/planner` with an `action`.
4. The route validates authentication, body size, origin, action inputs, and trip membership.
5. After a successful mutation, the client refreshes the entire planner state.

For a guest, `PlannerApp` uses `demoState`; attempting a mutation redirects to ChatGPT sign-in.

## Current data model

- `profiles`: identity display data keyed by email.
- `inventory`: personal gear owned by one profile.
- `trips`: trip metadata, owner, dates, and unique join code.
- `trip_members`: composite key of trip and member email; also snapshots display names.
- `needs`: shared trip checklist items, assignee, and packed state.
- `expenses`: a trip basket with payer, description, integer euro cents, and timestamp.
- `expense_participants`: participant snapshot for each expense. This prevents a later joiner from being charged for earlier baskets.

All monetary values are integer cents. Do not calculate or persist money with floating-point euro values.

## Expense settlement rules

- A camper can add a basket only as themselves; the server derives `paid_by_email` from authentication.
- A new basket is split across every current trip member and snapshots those emails.
- Only the payer can remove their basket.
- `calculateExpenseSummary()` assigns remainder cents deterministically by sorted participant email, then matches debtors to creditors into a compact repayment list.
- The total of all balances must equal zero cents.
- Currency is currently fixed to EUR. Do not add mixed-currency arithmetic without an explicit conversion or separate-ledger design.

## UI conventions

- Brand palette and responsive layout tokens live in `app/globals.css`.
- Desktop navigation is in the forest sidebar; mobile navigation is sticky below the top bar.
- Use Barlow Condensed for display headings and DM Sans for body text through CSS variables from `app/layout.tsx`.
- Keep touch targets, labels, disabled states, empty states, and narrow mobile layouts.
- Avoid introducing a component library unless the existing design can no longer support the requested interaction.
- Maintain translations in the `translations` object and server error translations in `ltErrors`.

## Security invariants

- Trust only the `oai-authenticated-user-*` headers read by `getChatGPTUser()`.
- Retain same-origin rejection for API writes.
- Use prepared D1 statements and one SQL statement per `prepare()` call.
- Check `isMember(tripId, user.email)` before trip-scoped mutations.
- Bound request bodies and clean all text.
- Never expose D1 bindings, credentials, internal ids, or authentication headers to the client.
- Preserve the baseline security headers set in `worker/index.ts`.

## Validation

- `npm run db:generate`: generate SQL after schema changes.
- `npm run lint`: lint TypeScript and React.
- `npm test`: production build plus rendered-product checks.
- Business logic tests use Node's test runner under `tests/`.
- Inspect generated migration SQL before deployment.

When this map becomes stale, update it in the same change as the architecture it documents.
