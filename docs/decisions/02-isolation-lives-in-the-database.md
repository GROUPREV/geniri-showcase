# Isolation lives in the database

**Status:** in force. **Owners:** the engineering team.

## Context

Every row in the platform belongs to someone: a person's own account, or a team
account with members and roles. The first version of the application checked
access in route handlers, the way most applications start. That works until a new
query path forgets the check, or a server action is added by someone who did not
read the first one.

The data is commercial as well as personal. A generation, a balance and an
invoice all belong to an account, and any of them leaking across accounts is a
serious failure, not a bug.

## Decision

Authorization is row-level security in PostgreSQL. A policy on each table asks the
database whether the current user belongs to the team account that owns the row,
and whether that member holds the permission the operation needs.

- The permission function reads staff roles from a table, in an authenticated
  session (`aal2` for the roles that need a second factor), rather than trusting a
  claim in the token. Revoking a role applies to the next query, not the next
  login.
- Policies are additive. A super administrator has its own policy; it does not
  switch the others off. That keeps the audit trail honest, because the audit
  trigger runs for everyone.
- Administrative functions write their audit row in the same transaction as the
  change they describe.

## Consequences

- The application's service role bypasses policies by definition, so it is only
  ever used on the server, and nothing user-facing is allowed to hold it.
- Every new table needs a policy and a test before it can ship. That is slower
  than a route check and it fails in a place a reviewer can see.
- The tests are pgTAP suites against a real PostgreSQL: who may read, write and
  delete, including the negative cases, run in CI on every database change.
- A permission bug is a database migration, not a deploy of application code.

## What we would reconsider

Nothing structural. If the application ever needed a second datastore with its own
access model, the same rule would have to be restated there rather than assumed to
carry over.
