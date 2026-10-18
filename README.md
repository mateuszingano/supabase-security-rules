# Supabase security rules for AI coding agents

Drop-in rules that make Cursor, Claude Code, Windsurf and other AI coding agents write **secure** Supabase code by default: Row Level Security on every table, policies scoped to the user, explicit grants, and the `service_role` key kept out of the browser.

AI tools scaffold Supabase apps fast — and often ship a table with no RLS, a `using (true)` policy, or the master key in the frontend. These rules encode the correct pattern so your agent stops doing that. Especially relevant now: on **Oct 30, 2026** Supabase stops auto-exposing new tables to the Data API, so every migration needs explicit grants.

## Install

Pick the file your tool reads, or just copy `AGENTS.md` — Cursor, Windsurf and Claude Code all read it.

| Tool | Where it goes |
|---|---|
| **Any (recommended)** | `AGENTS.md` at the repo root |
| **Cursor** | `.cursor/rules/supabase-security.mdc` |
| **Claude Code** | `CLAUDE.md` (or append to yours) |
| **Windsurf** | `.windsurf/rules/supabase-security.md` |
| **Older Cursor/Windsurf** | `.cursorrules` / `.windsurfrules` |

```bash
# fastest: drop the cross-tool file into your repo
curl -o AGENTS.md https://raw.githubusercontent.com/mateuszingano/supabase-security-rules/main/AGENTS.md
```

Or copy the relevant file from this repo. Commit it so your whole team's agents share the same rules.

## What the rules say

See [`AGENTS.md`](AGENTS.md) for the full text. In short, when the agent touches a `public` table it must, in the **same migration**: enable RLS, add a per-operation policy scoped to `auth.uid()` (or team membership), and `grant` explicitly. It must never ship the `service_role` key to the browser, never use `using (true)` on a client-reachable table, and never "fix" a `42501` with a blanket `grant ... on all tables`.

## Verify (the rules can't enforce themselves)

Agent rules are advisory — the agent can ignore them. Prove your schema is actually safe:

- **CI:** [`npx airlock-migrate`](https://www.npmjs.com/package/airlock-migrate) fails the build when a migration creates a table without RLS.
- **Live DB:** `SUPABASE_DB_URL=... npx airlock-rls` connects with the anon key and proves whether a table is readable without logging in.
- **Deployed app:** [the free scanner](https://shipsealed.com/service-role-scanner/) checks your live site for a leaked key or tables anyone can read — no signup.

More Supabase RLS guides: [shipsealed.com/rls](https://shipsealed.com/rls/).

## License

MIT. Use it, copy it, change it. Maintained by [ShipSealed](https://shipsealed.com). Not affiliated with Supabase.
