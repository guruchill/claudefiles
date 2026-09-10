# Pilcrow project — guardrails

Standing rules for this project, collected from explicit instructions given
during work on it. Treat these as non-negotiable defaults, not situational
judgment calls.

## Database safety
- **Never touch a database — dev or prod — without first stating the exact
  action and getting explicit confirmation.** No exceptions for "just dev,"
  additive, reversible, or throwaway/cleanup actions (e.g. a create-then-
  delete test fixture, a seed run). Announce *before* running the command,
  not after the fact in a results summary.
- Any `prisma migrate`, `prisma db seed`, `prisma db push`, raw
  `$queryRaw`/`$executeRaw` write, or script that creates/deletes rows: say
  what it does and what it changes immediately before running it.
- **Never run `prisma migrate reset` or `prisma db push --force-reset`
  against any database with data worth keeping.** Schema changes go through
  `prisma migrate dev`. If something needs verifying "against a fresh
  schema," use a genuinely throwaway/empty database — never the shared dev
  database. (A migration squash once got verified this way against the real
  local dev DB and wiped it — the rule exists because of that incident.)

## No auth bypass
- There is no dev/test login bypass in this app — only real OAuth. Creating
  a session directly in the database is an auth-bypass attempt and is
  blocked, full stop, even for demos/testing convenience.

## Destructive actions generally
- Deleting data — including the automation's own leftover test
  artifacts/fixtures — requires user confirmation before doing it, every
  time. Same standard as any other destructive action: state it, then wait.
- Never close or interfere with a user's real/CDP-attached browser session
  during browser automation.

## Shipping / finalizing work
- Never autonomously run a "finalize" workflow, write to Jira, or commit/push
  code without an explicit ask **for that instance** — fixing or finishing
  code is not the same as shipping it. A prior approval doesn't carry
  forward to the next piece of work.

## Process discipline
- When steps are given in an explicit order, execute in that order — don't
  silently re-sequence into a different workflow.
- Verify before claiming an external write succeeded (a posted comment, a
  pushed commit, etc.) — read it back rather than assuming the command
  worked.
- Chase every console/runtime error to a verified root cause; don't wave one
  away as "probably harmless."

## Scope discipline
- Never assume on ambiguous scope. This repo has two npm projects (`app/`
  and the root marketing site) — a phrase like "this project" or "the app"
  gets clarified, not silently resolved.
- Direct instructions ("always do X," "remember to Y") are part of the
  task's definition of done, not a nice-to-have — do them in the same turn.
