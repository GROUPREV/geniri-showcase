# One repository, many deployables

**Status:** in force since the platform was built. **Owners:** the engineering
team.

## Context

Geniri ships more than a website. There is the Next.js application customers use,
a component gallery the team builds interfaces in, an After Effects panel, a
DaVinci Resolve plugin and an end-to-end test application. Eighteen packages sit
under them: the component library, the i18n layer, the Supabase clients, billing,
mail, monitoring, policies and the extension runtime.

The desktop integrations present Geniri's results inside two video tools. They do
not generate anything themselves; they read the same data through the same
authorisation and draw it with a smaller version of the same component library.
Splitting them out would mean publishing shared packages to a registry, or
copying components and watching them drift.

## Decision

Keep everything in one pnpm workspace, built with Turborepo, versioned together.

## Consequences

- A change to a shared component reaches the web application and both desktop
  integrations in one commit, and the type checker proves it.
- The install is heavy and the build graph is real. CI cannot run everything on
  every change, so it decides what a change can still affect — see
  [CI minutes are a budget](03-ci-minutes-are-a-budget.md).
- The desktop applications build in their own formats (a signed extension package,
  a plugin bundle). They stay in the repository and are excluded from the web
  application's build, rather than living in a second project.
- Releases are coordinated. When the plugin and the application disagree about a
  payload, the fix lands in the shared package and both come up together.

## What we would reconsider

If the desktop tools grew their own roadmap and their own release rhythm, the
shared packages would become a contract between two products, and a registry with
versions would be the honest answer. That has not happened, and inventing the
machinery early would cost more than it saves.
