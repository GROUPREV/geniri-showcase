# A row-level security policy

An excerpt from one migration. It grants a staff role the ability to change the
site's media tiles, and shows the shape every policy in the schema follows.

```sql
-- D-89: the staff roles. `media` changes the tiles (not a built-in one's
-- existence: a built-in tile is switched off, never deleted), `marketing`
-- reads them. Each asks `has_staff_role`, read from staff_roles in an aal2
-- session, never a claim; the policies above stay as they are.

drop policy if exists "Staff roles read every site tile" on public.site_tiles;
create policy "Staff roles read every site tile"
  on public.site_tiles
  for select
  to authenticated
  using (public.has_staff_role('media') or public.has_staff_role('marketing'));

drop policy if exists "Media staff add site tiles" on public.site_tiles;
create policy "Media staff add site tiles"
  on public.site_tiles
  for insert
  to authenticated
  with check (public.has_staff_role('media'));

drop policy if exists "Media staff edit site tiles" on public.site_tiles;
create policy "Media staff edit site tiles"
  on public.site_tiles
  for update
  to authenticated
  using (public.has_staff_role('media'))
  with check (public.has_staff_role('media'));

drop policy if exists "Media staff remove the tiles staff added" on public.site_tiles;
create policy "Media staff remove the tiles staff added"
  on public.site_tiles
  for delete
  to authenticated
  using (public.has_staff_role('media') and not builtin);
```

## What to notice

- **The permission is a database question.** `has_staff_role` reads the roles
  table in an authenticated session, so removing a role takes effect on the next
  query. A token claim would last until the token expired.
- **Read, write and delete are separate policies.** A role that may edit is not
  silently a role that may delete, and the delete policy carries an extra
  condition: staff can remove a tile they added, never one that shipped with the
  site. That rule is in the database because that is where it can be enforced.
- **`with check` is not decoration.** On `update` it is what stops a permitted
  row from being moved somewhere the role does not belong. On `insert` it is the
  only check there is.
- **The migration is idempotent.** `drop policy if exists` then `create policy`,
  so applying it twice is safe and a failed run is repairable.
- **Additive, not overriding.** These policies add access for two staff roles; the
  super administrator's policy stays as it was. Nobody's access is widened by a
  side effect.
- **It is tested.** The pgTAP suites assert the positive case and the negative
  one: the role that may read but not delete, the account that sees no rows at
  all. [Isolation lives in the database](../docs/decisions/02-isolation-lives-in-the-database.md)
  explains why this sits here rather than in a route handler.
