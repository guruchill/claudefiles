# Pilcrow — product & architecture context

This document exists to give an AI assistant (Claude Desktop) full context on
**Pilcrow**, an ADHD-friendly notes/lists app, so it can discuss the product,
its data model, and its codebase without needing the repo open. It is
descriptive, not a style guide — it does not tell the assistant how to write
code for this repo (that's `CLAUDE.md`, which stays in the repo itself).

Repo name on disk: `diacratic`. Product name: **Pilcrow**. Production domain:
`thepilcrow.app`.

## One-line pitch

An ADHD-friendly notes and lists app: snippets (lists, notes, images, links)
live in a flowing, resizable, draggable feed, organized with tags, chains,
and "laundry pile" collections — built by a solo developer (diagnosed ADHD at
50) around how a scattered brain actually works, rather than imposing GTD/PKM
structure up front.

Design philosophy (from the marketing site's About page): the app doesn't
send reminders that get lost in notification noise — it visually "lights up"
notes and lists that need attention instead. It doesn't enforce structure,
but can bring structure when needed (tags, chains, piles).

## Core concepts / vocabulary

- **Snippet** — the atomic unit of content. Four types: **List** (items with
  status: unstarted/in-progress/complete/cancelled), **Note** (Markdown),
  **Image** (uploaded, encrypted at rest), **Link** (URL with an
  auto-fetched preview: title/description/image).
- **The feed** — a responsive, multi-column, Pinterest/masonry-style stream
  (not a rigid grid). Snippets can be dragged to reposition, resized by
  dragging the bottom edge, pinned to the top, and collapsed to just a
  name (optionally showing tags/creation date).
- **Tags** — user-defined, colored labels attachable to any snippet; used to
  filter the feed.
- **Chaining** — attaching one snippet underneath another ("master") so it
  indents, narrows, and moves/collapses together with its parent. Single
  level only — chaining a snippet that's already chained re-parents it to
  the top of that chain rather than nesting further.
- **Laundry piles** — custom, named views over a subset of snippets, without
  removing them from the main feed. A snippet keeps independent
  collapse/position/order state per pile. Every user has one permanent,
  un-deletable, un-renamable **default pile ("All Snippets")** that contains
  every snippet automatically. Piles can be **archived** (dropped from the
  tab row) and have a **master list** that rolls up every list item from
  every List snippet inside the pile, with its own hide-completed/hide-
  cancelled toggles.
- **Light-up** — a snippet visually highlighted (outline color) to catch the
  eye. Two independent mechanisms: **manual** light-up (free — pick a color
  per snippet from a menu) and **scheduled** light-up dates (Pro — fixed
  date, or recurring weekly/monthly/annually).
- **Item linking** — a list item (or a snippet generally) can be linked to
  any other snippet, cross-referencing content laterally outside the
  chain/pile hierarchy.
- **Getting Started pile** — one special, admin-owned shared pile every new
  user is automatically given view access to; it's the one shared pile a
  member can never fully leave (they can only hide it from their own tab
  row).

## Feature matrix

### Free tier
- All four snippet types, unlimited use within quota
- The feed: drag-to-reposition, drag-to-resize, pin, collapse
- Tags with custom colors, tag-based filtering
- Chaining
- Laundry piles (quota: **5 piles / 100 snippets** on the seeded free tier —
  quotas are actually per-`Tier` DB rows, admin-configurable, not hardcoded)
- Full-text search, date-picker jump-to-day, tag filter — all in the header
- Manual light-up (per-snippet color highlight)
- Sign in with Google, Microsoft, or Apple (no separate password)
- Every new signup gets a **30-day Pro trial** before falling back to free

### Pro tier ($10/mo or $100/yr, billed via Paddle + RevenueCat)
- Unlimited piles and snippets
- **Light-up dates** — scheduled (fixed/recurring) light-up, not just manual
- **Shared piles** — invite others into a pile via single-use invite links;
  free users can view a shared pile, Pro users can contribute to it. Pile
  owner can set a member to read-only (`canContribute`) independent of that
  member's own tier.
- **Laundry pile webhooks** — each pile can expose a webhook URL; POSTing to
  it creates a Note snippet in that pile. Three webhook modes: `STANDARD`
  (plain `{title?, text}` body), `POCKET` (heypocket.com voice-recording
  webhook envelope — only acts on `summary.completed`, updates the same
  snippet in place on later revisions rather than duplicating), and `JIRA`
  (Jira Server/Data Center webhook envelope — `jira:issue_created` /
  `jira:issue_updated`, same update-in-place behavior). A webhook can apply
  a default tag to everything it creates.
- **MCP server** — point an MCP-compatible client (e.g. Claude) at the
  account via a personal API key and let it read/create/organize snippets:
  create snippets, attach tags, send snippets to piles, manage piles/tags,
  set light-up. (Tool names as implemented: `create_snippet`,
  `append_to_snippet`, `update_snippet`, `get_snippet`, `list_snippets`,
  `create_pile`, `update_pile`, `list_piles`, `send_snippet_to_pile`,
  `create_tag`, `attach_tag`, `list_tags`, `set_manual_light_up`,
  `set_auto_light_up_color`, `set_light_up_schedule`,
  `remove_light_up_schedule`.)
- **PDF export** — export any laundry pile as a printable/portable PDF
- **Browser extension** (Chrome, Manifest V3) — send the current tab to a
  pile as a Link snippet, using its own API-key auth scheme (separate from
  MCP keys)
- **LCARS mode** — a novelty Star Trek-console visual skin, independent of
  the normal theme/font pickers, toggled from Settings > Advanced

### Native mobile app
An iOS app exists (see `MobileSession` model / `lib/mobile-apple.ts`),
authenticating via its own opaque bearer-token session type rather than the
web's cookie session or the MCP/extension API keys — separate from the three
premium-gated integration surfaces above.

### Personalization (all tiers)
- 7 color palettes (Dark Green default, plus a "][" shaded variant, Dark/
  Light Amber, Dark/Light Blue, plain Light) — pure CSS custom-property
  swaps, no per-component theming
- 13 selectable fonts, from coding monospace (IBM Plex Mono default,
  JetBrains Mono, Fira Code) to typewriter faces (Courier Prime, Special
  Elite, Cutive Mono) to serif/sans options (Libre Caslon Text, Libre
  Baskerville, Manrope, Space Grotesk, Urbanist, Roboto Thin, Lobster Two)
- Font size preference, show/hide snippet timestamps, tag-filter scoping
  (show only tags used in the current pile), chained-snippet
  expand/collapse-inheritance behavior — all per-user settings

### Admin capabilities
The very first person ever to sign in to a Pilcrow instance is automatically
made an admin (no separate provisioning step). Admin screens
(`(app)/admin/*`):
- **Signups** — signup mode (open / allowlist / closed) and the allowlist
  itself
- **Users** — view/manage accounts, assign tiers
- **Tiers** — create/edit tiers, their quotas (max piles/snippets, deleted-
  item retention days), and which `PREMIUM_FEATURES` they unlock (tier→
  feature grants are admin-managed *data*, not code — adding a tier or
  changing its entitlements never needs a deploy)
- **Exports** — audit log of "export all snippets as XML" requests (who,
  when, from where — not the exported content itself)
- **Bug reports** — an in-app bug-report button (admin-toggleable; off by
  default since it files real Jira issues) and their submissions
- **Contact** — messages submitted through the marketing site's contact form
- **Stats** — app-wide usage stats
- App-wide settings: billing kill-switch (pause self-serve upgrades without
  touching billing config), signup webhook (notify an external URL on every
  new signup)

## Architecture

Two independent Node projects sharing one origin, split by path:
- **`/` (repo root)** — Astro marketing/info site (`src/`, `astro.config.mjs`)
- **`/app`** — the actual product: Next.js App Router + TypeScript, Postgres
  via Prisma, Auth.js v5 (Google/Microsoft/Apple), Tailwind, React Query

Local dev: `scripts/dev-proxy.mjs` fronts Astro (4321) and Next (8801).
Production: one Container App/image — `app/server.js` is a hand-written
server that serves the Astro static build at `/` and hands off to Next's
handler at `/app` (this is why the app doesn't use Next's `output:
"standalone"`).

### Data model highlights (`app/prisma/schema.prisma` is the source of truth)
- **Snippet** base table + one 1:1 detail table per type (`ListSnippetDetail`,
  `NoteSnippetDetail`, `ImageSnippetDetail`, `LinkSnippetDetail`) — not a JSON
  blob — so adding a 5th type is additive (new enum value + detail table +
  component), no migration of existing rows.
- The default "All Snippets" pile is a real `LaundryPile` row
  (`isDefault: true`), not special-cased in code. One `SnippetPileMembership`
  join model drives per-pile position/collapse/pin/height/column/columnSpan
  uniformly for every pile, default included.
- Chaining is `Snippet.parentId` self-relation, single-level by construction
  (enforced in the chain API route, not the schema).
- Tiers (`Tier` table) are admin-managed data with a `rank`, quotas, and
  optional `revenueCatEntitlementId` (billing-backed vs. admin-comped).
  Which features a tier unlocks lives in `TierFeature` (data), while the
  *catalog* of feature keys that can exist at all lives in code
  (`config/tiers.ts`'s `PREMIUM_FEATURES`).
- Soft delete: a deleted snippet moves to a "Deleted" state (`deletedAt`) and
  is purged after `Tier.deletedRetentionDays`.
- Sensitive content (email, provider account IDs, tag names, note/list/link
  text, image bytes) is encrypted at rest per-owner (AES-256-GCM,
  `lib/crypto.ts`), with deterministic "blind index" hash columns
  (`emailHash`, `nameHash`, etc.) alongside the ciphertext so equality
  lookups and uniqueness constraints still work without ever storing
  plaintext.
- Read-only locking: a snippet can be individually locked (`readOnly`), or
  implicitly locked because *any* pile it belongs to is locked
  (`LaundryPile.readOnly`) — resolved through one chokepoint function so the
  two sources of truth never diverge.

### Feed rendering
`MasonryGrid` computes its own column placement (a greedy shortest-column
packer) rather than using CSS multi-column or a masonry library, because
dnd-kit requires each column to be its own `SortableContext` for cross-
column drag to work. Column layout is derived/recomputed client-side, not
persisted — only the flattened order and last-dropped column are persisted,
so a resize on one device reflows correctly everywhere without needing to
sync layout state.

### MCP server implementation
`src/lib/mcp/server.ts` builds a fresh `McpServer` per request, tools scoped
to one `userId` by closure; `api/mcp/route.ts` authenticates via a hashed
`McpApiKey` bearer token (never the session cookie), gates on the
`mcp-server` premium feature, and runs in fully stateless HTTP mode (no
in-memory session state to survive between requests).

### Auth
Auth.js v5, database-session strategy (not JWT), providers: Google,
Microsoft, Apple. `auth-adapter.ts` atomically sets a new user's tier,
first-user-becomes-admin, and default pile on creation. Middleware runs on
the Node runtime (not Edge) because resolving a database session needs a
real Prisma query.

### Billing
RevenueCat + Paddle. Subscription events land as `RevenueCatWebhookEvent`
rows (idempotent by `eventId`) and sync into the same `tierId`/`tierEndDate`
columns used for admin-granted temporary upgrades — one mechanism for both
paid and comped tier changes.

## Repo layout

```
diacratic/
├── src/, astro.config.mjs, package.json    # marketing site (Astro)
├── app/                                     # the Next.js product
│   ├── src/app/(auth)/                      # sign-in
│   ├── src/app/(app)/                       # feed, piles, profile, admin — behind auth
│   ├── src/app/api/                         # REST API + MCP server + pile webhooks
│   ├── src/components/                      # feed/, snippets/, tags/, piles/, header/, admin/, profile/
│   ├── src/lib/                             # Prisma client, auth, tier gating, crypto, MCP tools, link unfurling
│   ├── config/                              # app.ts, fonts.ts, theme.ts, tiers.ts
│   └── prisma/                              # schema, migrations, seed
├── extension/                                # Chrome MV3 extension (send tab to pile)
├── infra/                                    # Bicep IaC for Azure
└── docs/                                     # architecture.md, local-dev.md, deployment.md, revenuecat-billing.md
```

## Deployment

Azure: Container Apps + Postgres Flexible Server, provisioned via Bicep
(`/infra`) and deployed by a GitHub Actions workflow. Image bytes for
Image snippets live encrypted in Postgres, not blob storage — a one-time
migration off Azure Blob Storage has already run and that storage account
has been decommissioned (a legacy `storageKey` column remains, inert).
