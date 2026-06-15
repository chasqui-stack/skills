---
name: chasqui-deploy
description: "INVOKE when deploying a Chasqui stack to a server, or operating it day-2 (logs, ship a new version, rollback, shell in). Chasqui deploys with Kamal 2, one service at a time, in a strict order. Covers the per-service deploy, the core→gateway→admin ordering, and the day-2 commands."
---

# Deploy a Chasqui stack (Kamal 2)

Chasqui deploys each service with **Kamal 2** from its own `config/deploy.yml`.
The services are deployed **independently** but in a **strict first-install
order**, because of health checks and CORS.

> Source of truth — read it; it has the exact per-service config edits and the
> full prerequisites:
> https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.2.4/docs/DEPLOY.md

## Prerequisites

Kamal is a Ruby gem — it must be installed on your machine before deploying:

```bash
gem install kamal      # needs Ruby; this is Kamal 2 (https://kamal-deploy.org)
```

You also need: a VM you can SSH into (Docker is installed by `kamal setup`), DNS
`A` records for `api.`, `wsp.` and `admin.<domain>` → the server IP, and a Docker
registry account (Docker Hub works). See DEPLOY.md for the full list.

## Order matters on first install: core → gateway → admin

1. **Core first.** Edit `core/config/deploy.yml` (server IP, registry username,
   host, the non-secret LLM/embedding provider+model env, `CORS_ORIGINS=https://admin.<domain>`),
   then `kamal setup`/`kamal deploy`.
2. **Gateway** (whatsapp/telegram). Set `CORE_URL=https://api.<domain>` — the
   gateway health-checks against a reachable core, so the core must be up first.
3. **Admin.** Set the build arg `VITE_API_BASE_URL=https://api.<domain>` — and the
   core's `CORS_ORIGINS` must already allow `https://admin.<domain>`.

The reasons (health check + CORS) are why the order is not optional. Read DEPLOY.md
for each service's full config block before editing.

## Day 2

```bash
kamal app logs -f        # follow logs (alias: logs)
kamal deploy             # ship a new version
kamal rollback <version> # revert to a previous release
kamal app exec -i bash   # shell into the container (alias: app-terminal)
```

## Conventions to respect

- **Secrets** go through Kamal's secrets mechanism — never commit `.kamal/secrets`
  or `.env`. Only the non-secret provider/model/host env lives in `deploy.yml`.
- **Gateway-local fallback strings** (`ERROR_REPLY`/`UNSUPPORTED_REPLY`) are set per
  deployment in the gateway's env, in the users' language.
- The generated project ships a `docker-compose.yml` for collaborators; Kamal is
  the path to a real server.

## Anti-patterns

- ❌ Deploying the gateway or admin before the core — health check / CORS will fail.
- ❌ Committing secrets — Kamal secrets only.
- ❌ Pointing the admin/gateway at the wrong `api.<domain>` — the build arg and
  `CORE_URL` must match the deployed core host.
