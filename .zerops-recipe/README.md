# Tasklight — zcp-produced Full-stack App

<!-- #ZEROPS_EXTRACT_START:intro# -->
A full-stack task manager scaffolded by [zcp](https://zerops.io) — [SvelteKit](https://svelte.dev/docs/kit) + [PostgreSQL](https://www.postgresql.org/) + [Valkey](https://valkey.io/), running on [Zerops](https://zerops.io). It ships the whole development lifecycle as ready-made environments: an AI-agent workspace, remote (CDE) and local development, and small + highly-available production.
<!-- #ZEROPS_EXTRACT_END:intro# -->

⬇️ **Full recipe page and deploy with one-click**

[![Deploy on Zerops](https://github.com/zeropsio/recipe-shared-assets/blob/main/deploy-button/light/deploy-button.svg)](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=small-production)

This is what a **zcp-produced own-code project** looks like as a GitHub recipe: the app code and the `.zerops-recipe/` definition live in the same repo, so cloning it gives you both the app and every environment it knows how to run in. The first two environments are **control-plane** environments — an AI agent workspace and a remote CDE — followed by local dev and two production tiers.

- **AI Agent** [[info]](/0%20%E2%80%94%20AI%20Agent) — [[deploy with one click]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=ai-agent)
- **Remote (CDE)** [[info]](/1%20%E2%80%94%20Remote%20(CDE)) — [[deploy with one click]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=remote-cde)
- **Local** [[info]](/2%20%E2%80%94%20Local) — [[deploy with one click]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=local)
- **Small Production** [[info]](/4%20%E2%80%94%20Small%20Production) — [[deploy with one click]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=small-production)
- **Highly-available Production** [[info]](/5%20%E2%80%94%20Highly-available%20Production) — [[deploy with one click]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=highly-available-production)

## Take-over guide

<!-- #ZEROPS_EXTRACT_START:takeover-guide# -->

### Make it yours
Clone this repo, push it to your own GitHub, and connect your copy in the Zerops service detail (**Build & deploy pipeline → connect repository**). Every `git push` to your branch then builds and deploys. The app is yours from that point — upgrade by editing and pushing, not by re-importing.

### Promote across environments
Start in the **AI Agent** or **Remote (CDE)** environment while you build, then import **Small Production** when you're ready to ship. The import yamls differ only in sizing and high-availability — the app code is identical across all of them.

### Run it on your domain
Add your domain under the project's **Public access & domains**, point a CNAME at the assigned target, and Zerops issues TLS automatically.

### Backups
Production databases are backed up daily. Restore from the **Backups** tab in the database service detail.

<!-- #ZEROPS_EXTRACT_END:takeover-guide# -->

## Knowledge Base

<!-- #ZEROPS_EXTRACT_START:knowledge-base# -->

### Sessions in Valkey
In the production tiers, SvelteKit sessions and the rate-limit counters live in Valkey rather than Postgres, so app containers stay stateless and scale horizontally without sticky sessions. The `CACHE_URL` connection string is derived from the `cache` service hostname at import time.

### Database connection pooling
Each app container opens its own pool. When you raise `minContainers` or autoscaling kicks in, watch the database's connection ceiling — the HA tier uses a larger Postgres so the aggregate pool fits comfortably.

<!-- #ZEROPS_EXTRACT_END:knowledge-base# -->

---

For more examples see all [recipes](https://app.zerops.io/recipes) on Zerops. Need help? Join the [Zerops Discord community](https://discord.gg/zeropsio).
