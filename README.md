# Geniri

Geniri is a generative media platform for working creatives, built by REVOLUTION Group and live at [geniri.io](https://www.geniri.io). Customers buy credits and generate images, video, music and voice from a catalogue of third-party models. The platform meters the credits, dispatches the job, stores the result, and refunds automatically when a generation fails. Payment is by CCP transfer — BaridiMob or at the post office — against a `GNR-…` reference, reviewed by a person in an admin queue, with a French invoice issued on confirmation.

This repository is the record we submitted to the **ThirdUni AI programme** (September 2026). It explains how the platform is built, what it is built with, the decisions that shaped it, and shows a few short excerpts of the code. The product's source stays private. [NOTICE.md](NOTICE.md) says exactly what is withheld and why.

- Product: [www.geniri.io](https://www.geniri.io)
- Walkthrough video: being recorded; the link lands here before the deadline
- Programme record: [docs/program.md](docs/program.md)
- Contributors and roles: [CONTRIBUTORS.md](CONTRIBUTORS.md)

## Where each submission item is

| # | Item | Answer |
| --- | --- | --- |
| 1 | Name and one-line summary | The title and the first paragraph |
| 2 | Description | [What a customer does](#what-a-customer-does) |
| 3 | Live demo | [geniri.io](https://www.geniri.io) — free account, 2,500 credits, no card |
| 4 | Code repository | This repository: [architecture](ARCHITECTURE.md), four decision records, two code excerpts |
| 5 | Demo video | Linked at the top of this page once recorded, before 31 October 2026 |
| 6 | Team and roles | [Team](#team-and-roles) below, and [CONTRIBUTORS.md](CONTRIBUTORS.md) |
| 7 | Tech stack | [The stack](#the-stack) |
| 8 | What happens next | [The last section](#what-happens-next) |

## What a customer does

A customer buys a credit pack, opens a creative surface, writes a prompt, chooses a model and a format, and generates. Video and music jobs run asynchronously: the job is queued, the provider is called, the result is stored, and the credit is charged for the job and returned if it fails. A failed generation is refunded without a support ticket, because the ledger reconciles the failure as part of the same flow that charged for it.

Administration is deliberate, not automated. Payments arrive by CCP transfer, by BaridiMob or at a post office, with a `GNR-…` reference. Someone opens `/admin`, matches the transfer to the order, and confirms it; confirmation is what issues the invoice and releases the credits. This is slower than a card checkout and it is the point: the customer is buying from a small studio, in a market where transfer is the normal instrument for a business purchase.

The platform speaks French, English and Arabic, and Arabic is treated as a direction as well as a language, not as translated text dropped into a left-to-right page.

## Try it

The product is live at [geniri.io](https://www.geniri.io). It opens in a browser and needs no install. Sign-up is open: [create an account](https://www.geniri.io/auth/sign-up) with an email address and the free plan starts with 2,500 credits, enough to make real generations rather than look at screenshots — and to watch the balance move as they land. No card, no call with a salesperson. Paid plans are priced in dinars and paid by CCP transfer, so a judge who wants to test a paid surface should ask and an account will be provisioned instead.

The source is private, so there is no clone-and-run in this repository. What a checkout takes, for the record: Node 20.19 or newer (22.13 and 24 also work), pnpm 10.19, and Docker for the local Supabase stack, then `pnpm install`, `pnpm supabase:web:start`, `pnpm supabase:web:reset` and `pnpm dev`. [ARCHITECTURE.md](ARCHITECTURE.md) says what each part does once it is running.

## The stack

| Layer | Choice | Version |
| --- | --- | --- |
| Web application | Next.js App Router, React | Next 16.3.3, React 19.2.8 |
| Styling | Tailwind CSS, local component library | Tailwind 4.3.3 |
| Data and auth | Supabase: PostgreSQL, auth, storage, row-level security | supabase-js 2.115.0 |
| Language | TypeScript across every app and package | 6.0.3 |
| Monorepo | pnpm workspaces, Turborepo | pnpm 10.19.0 |
| Translation | i18next, per-locale JSON, ICU placeholders | i18next 26.4 |
| Hosting | Vercel | — |
| Desktop integrations | After Effects panel (CEP), DaVinci Resolve plugin | — |

### AI tools used in building it

The team codes with agents: Claude Code, Gemini and JetBrains Junie, whose repository-level instructions live in the source tree. They do implementation and review work under human direction; the architecture, the release decisions and what ships are the team's. The product itself calls third-party model providers from the server, and how it chooses and directs them is not published.

## How the pieces fit

There are two things to keep apart when reading this repository: the **product surface** a customer touches, and the **administrative surface** that decides who may touch what. Both read and write the same PostgreSQL database, and the database is where tenant and role isolation is enforced, not the application layer.

```text
apps/web            Next.js application: storefront, generation surfaces, account, admin
apps/dev-tool       component gallery used while building the interface
apps/after-effects  CEP panel so editors can use Geniri inside After Effects
apps/davinci-resolve  plugin for DaVinci Resolve
packages/*          eighteen shared packages: ui, next, i18n, supabase, billing,
                    cms, policies, mailers, monitoring, analytics, otp, features,
                    shared, mcp-server, extension-core, extension-ui,
                    database-webhooks, email-templates
supabase/           420 ordered SQL migrations, seeds, pgTAP tests
```

[ARCHITECTURE.md](ARCHITECTURE.md) goes one level deeper: the request path for a generation, the credit ledger, the admin review, and how the pieces are tested. What it does not do is describe how jobs are routed to models or how prompts are built. That layer is the product's own work and stays private.

## What the programme produced

The four weeks did not produce a greenfield demo. Djaber Djarri joined as a participant and worked inside the running platform; Pius Sunday Ojwo led the engineering. The output was a set of changes that shipped to production and the release now in front of it: **v0.4**, which is in internal alpha testing before it moves to beta, a protected sub-public test. There is no date committed for either stage, and this file will not invent one.

The walkthrough video and the Demo Day date are added here before the submission closes on 31 October 2026; judging runs 1 to 7 November. [docs/program.md](docs/program.md) records the programme as ThirdUni published it, what we did inside it, and what still has to be added.

## Team and roles

- **Djaber Djarri** — participant, ThirdUni cohort. The product side: what the platform promises, what the release contains, and the cases it has to handle before it ships.
- **Pius Sunday Ojwo** — software engineering lead. Architecture and implementation: the application, the database and its policies, the delivery pipeline, and the releases described in [docs/program.md](docs/program.md).

Further contributors are being confirmed and will be named, with one honest line each, before 31 October 2026. [CONTRIBUTORS.md](CONTRIBUTORS.md) holds the same list with more context.

## The decisions worth reading

Four decisions explain most of the shape of the platform:

1. [One repository, many deployables](docs/decisions/01-one-repository-many-deployables.md) — why the web app, the two desktop integrations and eighteen packages share a repository, and what that costs.
2. [Isolation lives in the database](docs/decisions/02-isolation-lives-in-the-database.md) — authorization as row-level security with tests, rather than `if` statements in route handlers.
3. [CI minutes are a budget](docs/decisions/03-ci-minutes-are-a-budget.md) — the measurement that led to deciding, per pull request, which expensive checks can still find something.
4. [The invoice waits for a person](docs/decisions/04-the-invoice-waits-for-a-person.md) — why payment confirmation is a manual review rather than a webhook, and what breaks in each direction.

## Code excerpts

Two short ones, chosen because they show the work rather than the product:

- [Deciding what CI should run](examples/01-deciding-what-ci-runs.md) — the pure function that turns a pull request's changed files into a plan, and fails toward running everything.
- [A row-level security policy](examples/02-a-row-level-security-policy.md) — how a staff role is granted access to one table, read from the database rather than from a token claim.

## What happens next

v0.4 moves from internal alpha to a capped beta, then release. The beta is where the creative surfaces meet people who did not build them, so the first weeks are spent on what that exposes rather than on new features. In parallel: the generation queue under load, the admin review for the hours when one person is not at the desk, and documentation good enough for a second engineer to ship without asking. Success is a customer who renews without writing to support, and a release that does not need a hotfix in its first week.

## Licence

No licence is granted. The text and excerpts are published for evaluation of this submission; see [NOTICE.md](NOTICE.md).
