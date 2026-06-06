# Tasklight — zcp-produced Full-stack App

<!-- #ZEROPS_EXTRACT_START:intro# -->
**Tasklight** is a production-ready, full-stack task manager — [SvelteKit](https://svelte.dev/docs/kit) + [PostgreSQL](https://www.postgresql.org/) + [Valkey](https://valkey.io/) — scaffolded by [zcp](https://zerops.io) and ready to run on [Zerops](https://zerops.io) across the **entire development lifecycle**: an AI-agent workspace, remote (CDE) and local development, and small + highly-available production. Clone it as the starting point for your own app, or deploy it as-is to see a complete Zerops setup end to end.
<!-- #ZEROPS_EXTRACT_END:intro# -->

⬇️ **Full recipe page and deploy with one-click**

[![Deploy on Zerops](https://github.com/zeropsio/recipe-shared-assets/blob/main/deploy-button/light/deploy-button.svg)](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=small-production)

Pick the environment that matches where you are in the lifecycle:

- **AI Agent** [[info]](/0%20%E2%80%94%20AI%20Agent) — [[deploy]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=ai-agent)
- **Remote (CDE)** [[info]](/1%20%E2%80%94%20Remote%20(CDE)) — [[deploy]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=remote-cde)
- **Local** [[info]](/2%20%E2%80%94%20Local) — [[deploy]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=local)
- **Small Production** [[info]](/4%20%E2%80%94%20Small%20Production) — [[deploy]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=small-production)
- **Highly-available Production** [[info]](/5%20%E2%80%94%20Highly-available%20Production) — [[deploy]](https://app.zerops.io/recipes/detail?github=https://github.com/fxck/recipe-sim-zcp-fullstack&yaml=highly-available-production)

## Take-over guide

<!-- #ZEROPS_EXTRACT_START:takeover-guide# -->
Deploying gives you a running copy on a Zerops subdomain. These steps turn that copy into **your** application — your repository, your domain, your release process.

### 1. Make it your own
Clone the recipe, push it to your own GitHub, and connect your copy so every push deploys.

```bash
git clone https://github.com/fxck/recipe-sim-zcp-fullstack my-tasklight
cd my-tasklight
git remote set-url origin git@github.com:<you>/my-tasklight.git
git push -u origin main
```

In the Zerops service detail open **Build & deploy pipeline → connect repository**, point it at your fork, and pick a branch. From now on a `git push` builds and deploys automatically.

> [!NOTE]
> Prefer the GitHub UI? Hit **Use this template** on the repo to create your own copy in one click, then connect it the same way.

### 2. Point it at your domain
By default the app answers on a generated `*.zerops.app` subdomain. To run on your own:

1. Open the project's **Public access & domains**.
2. Add your domain and copy the assigned CNAME target.
3. Create the CNAME record at your DNS provider.

Zerops provisions and renews TLS automatically — no certificate management.

### 3. Configure environment variables
The recipe wires the essentials for you; add your app's own variables in the service's **Environment variables** tab.

| Variable | Source | Notes |
|----------|--------|-------|
| `APP_KEY` | generated | Session/cookie encryption key, set at import. |
| `DATABASE_URL` | derived | Built from the `db` service connection at runtime. |
| `CACHE_URL` | derived | Built from the `cache` (Valkey) service. |
| `PUBLIC_BASE_URL` | you | Set once you've attached your domain. |

### 4. Scale for production
The two production environments differ only in sizing and high availability — the app code is identical.

```yaml
services:
  - hostname: app
    minContainers: 2      # always-on
    maxContainers: 6      # autoscales horizontally under load
    verticalAutoscaling:
      minRam: 1           # Zerops grows RAM/CPU within bounds
```

> [!TIP]
> Start on **Small Production** and switch to **Highly-available Production** when you need multi-node Postgres/Valkey and zero-downtime failover. Both import the same repository.

### 5. Back up and restore
Production databases are backed up daily. Restore a snapshot from the **Backups** tab in the database service detail — point-in-time for HA Postgres.

### 6. Monitor and debug
Per-service **Runtime logs** and **build logs** are in the service detail; the project **Statistics** service tracks CPU/RAM/requests. SSH into any container from its detail page for live debugging.

### 7. Upgrade
Because you own the repo, upgrades are just edits: bump dependencies, commit, push — your pipeline rebuilds.

> [!WARNING]
> Schema changes run via `prisma migrate deploy` in the run-init hook. Test migrations against a stage or remote-CDE environment before pushing to production.
<!-- #ZEROPS_EXTRACT_END:takeover-guide# -->

## Knowledge Base

<!-- #ZEROPS_EXTRACT_START:knowledge-base# -->
### Architecture
Stateless SvelteKit containers sit behind the Zerops L7 balancer; all shared state lives in Postgres and Valkey, so the app scales horizontally without sticky sessions.

![Tasklight architecture](https://raw.githubusercontent.com/fxck/recipe-sim-zcp-fullstack/main/.zerops-recipe/architecture.svg)

### How sessions and caching work
In the production tiers, SvelteKit sessions and rate-limit counters live in **Valkey** rather than Postgres. That keeps app containers stateless — any container can serve any request, which is what makes horizontal autoscaling safe. The `CACHE_URL` connection string is derived from the `cache` service hostname at import time.

### Database connection pooling
Each app container opens its own connection pool. When `minContainers` rises or autoscaling kicks in, the *aggregate* pool grows with it — so the HA tier ships a larger Postgres whose connection ceiling comfortably fits the maximum container count. If you raise `maxContainers`, check the pool math.

### Troubleshooting
- **502 right after deploy** — the app booted before migrations finished. Migrations run in the run-init hook; give it a few seconds, or check build logs for a failed `prisma migrate deploy`.
- **Sessions dropping under load** — confirm `CACHE_URL` resolves and the `cache` service is healthy; without Valkey, sessions fall back to per-container memory and break across containers.
- **Slow cold queries** — the single-node Small Production Postgres is fine for moderate traffic; move to HA Production (or scale RAM) before heavy analytical queries.
<!-- #ZEROPS_EXTRACT_END:knowledge-base# -->

---

For more examples see all [recipes](https://app.zerops.io/recipes) on Zerops. Need help? Join the [Zerops Discord community](https://discord.gg/zeropsio).
