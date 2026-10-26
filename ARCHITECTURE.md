# Architecture

This is the map a judge or a new engineer needs before reading code. It describes
the deployed shape of Geniri as of October 2026, and the three paths that matter:
a generation, a payment, and a permission check. It stops short of the
instruction layer, which [NOTICE.md](NOTICE.md) explains.

## The deployment shape

One Next.js application, one PostgreSQL database, and four smaller applications
that share the same packages.

```text
Browser ──▶ Next.js on Vercel (dub1) ──▶ Supabase PostgreSQL
                    │                        │
                    │                        ├─ auth (sessions, MFA)
                    │                        ├─ storage (generated media)
                    │                        └─ row-level security
                    │
                    ├─▶ model provider APIs (server-side only)
                    └─▶ Resend (transactional mail)
```

The desktop integrations, an After Effects panel and a DaVinci Resolve plugin,
call the same surface as the web application. They ship from this repository and
reuse `packages/extension-core` and `packages/extension-ui`, so a change to how a
result is presented is written once.

Vercel runs the application in `dub1` and holds the scheduled work as cron routes:
twenty-eight of them, most of it housekeeping that has to happen whether or not
anyone is awake — reconciliation with providers, credit refills, renewal
reminders, pruning old notifications. The one a customer feels is the generation
queue, which runs every minute: it recovers dead dispatches, asks each provider
about every generation still in flight, reaps what is dead at a ceiling, and
moves waiting rows into the slots that just freed. It answers within its own
deadline of 280 seconds whatever a provider is doing, so one slow provider cannot
hold the slot. Every write it makes is conditional on the state it read, so a run
that overlaps the next one cannot act twice on one row. The routes are guarded by
a shared `CRON_SECRET`; there is no separate worker tier to operate.

## The request path for a generation

1. The browser posts a server action with the prompt, the surface and the chosen
   format. The session cookie identifies the user. Nothing about identity is
   trusted from the request body.
2. The server action checks the credit balance and the permission for that
   surface, writes a `generations` row in `queued` state, and records the credit
   movement it is about to make. The charge and the row are written together, so
   a job cannot exist without its charge.
3. Fast work, which is most image generation, is dispatched inline and returns a
   result. Long work — video, music, batches — returns the row, and the
   generation queue picks it up.
4. The provider call happens on the server. Provider keys never reach the browser,
   and the provider's answer is normalised before anything is stored.
5. The result is written to storage and the row moves to `completed`, or to
   `failed` with the provider's reason. A trigger on the transition to `failed`
   or `cancelled` writes the refund in the same transaction, and a unique index
   makes a second refund for the same generation impossible — which is why there
   is no separate refund queue.
6. Video jobs have their own table and their own reaper, because their ceilings
   and their failure modes are different. The reaper runs every fifteen minutes.

Batches follow the same shape at a different scale: the batch is pooled, each row
is enqueued separately, and the reconcile route decides when the batch is done.

## Credits and the ledger

Credits are not a number someone types into a user row. Every movement is a row in
`credit_transactions` with a reason and a generation reference where one exists,
and a database trigger carries the running total in `account_credits`. The balance
is therefore a cache of the entries, never a value that can move without a
transaction behind it. That makes the awkward questions answerable months later:
what was charged, what was returned, and why.

The admin review reads the same ledger. Confirming a transfer writes the transaction
through the same path as any other credit grant, so there is one way credits
appear and one place to look when a balance is disputed.

## Payment, invoice and the admin queue

Payment is by bank transfer. The customer receives a `GNR-…` reference with the
order; the transfer arrives with that reference; a person matches it in `/admin`
and confirms it. Confirmation releases the credits and issues the French invoice.

Two failure modes matter, because both happened:

- A transfer without a usable reference. The money is there and the match is
  manual, so unmatched transfers get their own view in the queue instead of
  disappearing into an inbox.
- A confirmation recorded twice. Invoice numbers come from one gapless database
  counter inside the confirming transaction, so a double click cannot burn a
  number or produce two invoices for one order.

## Tenancy and permissions

The platform is multi-tenant at the level of a team account, and every table that
belongs to one carries the column that says so. Authorization is enforced by
row-level security in PostgreSQL, not by checks in route handlers. A policy asks
the database two questions: is this user a member of this team account, and does
this member hold this permission. The permission function reads staff roles from a
table in an authenticated session rather than trusting a claim in the token, so
revoking a role takes effect on the next query rather than the next login.

Policies are additive. A super administrator's access is its own policy, not a
bypass of the others, and every function that performs an administrative action
writes its audit row in the same transaction.

The tests are the same shape as the enforcement: pgTAP suites run against a real
PostgreSQL and assert who may read, write and delete what, including the negative
cases.

## Translation and right-to-left

Three locales: French, English and Arabic. Every locale carries every key; a
guard fails the build when one is missing or an ICU placeholder differs, which is
also why the application does not load a fallback bundle for a French or Arabic
page. That fallback cost 225 KB on the slow profile before it was removed.

Arabic sets its direction on the subtree that carries it, and the direction
travels with the content rather than being patched in a stylesheet. The browser
tests cover the language switch as a user experiences it, because a flash of the
wrong language or direction is the failure people notice.

## Testing and delivery

The pipeline is one workflow with jobs that fail for different reasons:

| Job | What it proves |
| --- | --- |
| Guards | Repository rules that are cheaper to check than to review: file conventions, branch model, locale parity, required-check reporting |
| TypeScript and build | The whole tree typechecks and the Next application builds |
| Unit suites, sharded | Business logic and component behaviour |
| Database (pgTAP) | Migrations apply clean and the policies hold under a real PostgreSQL |
| Browser suites | Auth, card and image flows against a running application |
| Markdown lint | Documentation stays valid |

Expensive jobs are gated per change rather than run everywhere. [The CI
decision](docs/decisions/03-ci-minutes-are-a-budget.md) has the measurement and the one
rule that governs the gate. `development` is the integration branch, `main` is what
production serves, and a release is a reviewed pull request from one to the other.

## What this document does not describe

The instruction layer: how prompts are constructed, how models are chosen per
task, how output quality is measured per credit, and the benchmarking that feeds
it. That is Geniri's own work. The same restraint applies to provider commercial
terms, unpublished pricing, customer data, and anything in the administrative
surface that would help someone abuse it. [NOTICE.md](NOTICE.md) says it
plainly.
