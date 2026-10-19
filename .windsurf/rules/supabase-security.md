---
trigger: always_on
description: Supabase security — RLS, policies, grants, service_role
---

# Supabase security rules

When you create or change a table in a Supabase `public` schema, do ALL of the following in the **same migration**:

1. **Enable RLS** on the table:
   ```sql
   alter table public.<table> enable row level security;
   ```
2. **Add a policy per operation, scoped to the user** (not just to a role), unless the data is public on purpose (see below):
   - `SELECT` / `DELETE`: `using (user_id = (select auth.uid()))`
   - `INSERT`: `with check (user_id = (select auth.uid()))`
   - `UPDATE`: **both** `using (...)` and `with check (...)`

   For team/workspace data, scope by membership (the user's teams), not by role alone.
3. **Grant access explicitly.** Starting Oct 30 2026, new tables in `public` are **not** exposed to the Data API without a grant:
   ```sql
   grant select, insert, update, delete on public.<table> to authenticated;
   ```
   Grant to `anon` **only** for data that is genuinely public (a catalog, prices).
   If the id is `serial`/`bigserial` and the API inserts rows, also grant the sequence:
   `grant usage on sequence public.<table>_id_seq to authenticated;`

**Never:**
- Ship the `service_role` key (or any `sb_secret_...` key) to the browser, or put it in a `NEXT_PUBLIC_*` variable. It **bypasses RLS entirely** — server-side only.
- Use `using (true)` or `with check (true)` on private data a client role can reach. RLS is "on" but every row is exposed. For a table that is public on purpose (a catalog, pricing), a read-only `for select to anon, authenticated using (true)` is fine: say so in a comment and don't add write policies for those roles.
- "Fix" a `42501 permission denied for table` by running `grant ... on all tables in schema public`. That re-grants **every** table, including ones with RLS off, re-exposing them. Grant the one table instead.

**Good to know:**
- The `anon` (publishable) key is safe to ship to the browser **only because RLS limits it**. It is not a secret.
- RLS and grants are separate: a grant without RLS exposes every row; RLS without a grant returns a permission error before any policy runs.

**Verify before shipping (both free, MIT):**
- In CI: `npx airlock-migrate` fails the build when a migration creates a table without enabling RLS.
- Against the live database: `SUPABASE_DB_URL=... npx airlock-rls` connects with the anon key and proves whether a table is readable without logging in.
