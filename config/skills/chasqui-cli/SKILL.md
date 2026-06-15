---
name: chasqui-cli
description: "INVOKE when using the chasqui CLI to scaffold or extend a Chasqui stack: creating a new project (uvx chasqui new), running the setup wizard, pinning a stack version with --ref, or generating a tool module (chasqui generate module). Covers the commands, their options, and the provisioning flow."
---

# chasqui CLI

The `chasqui` CLI scaffolds and extends a Chasqui stack — *zero to a running AI
chat agent in one command*. It's published to PyPI; run it with `uvx` (no install
needed) or install it. New project creation and code generators à la
`rails generate`.

Authoritative reference (read for the release ceremony, version pinning, and the
generator contract):
→ https://raw.githubusercontent.com/chasqui-stack/cli/main/AGENTS.md
→ Stack architecture (what gets scaffolded): https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.2.4/docs/ARCHITECTURE.md

## Install / version

```bash
uvx chasqui --version        # run without installing → "chasqui X.Y.Z (stack vX.Y.Z)"
```

The CLI **pins a stack tag** (`STACK_TAG`): a given CLI release scaffolds the
services at that exact tag (ADR-005). That's why the version output shows both.

## `chasqui new <name>` — scaffold a project

```bash
uvx chasqui new my-agent
```

Runs: **preflight** checks (uv, node, postgres, …) → an interactive **wizard**
(LLM/embeddings provider, channels, secrets → `.env`s) → **scaffold** (downloads
the pinned stack) → **provision** (uv/npm install, createdb, migrate).

Options:

| Option | Effect |
|--------|--------|
| `--defaults` | Skip the wizard — placeholder config, CI-friendly. |
| `--channels whatsapp,telegram` | Pick channels non-interactively (overrides the wizard; pairs with `--defaults`). |
| `--skip-provision` | Write files only; no uv/npm/createdb/migrate. |
| `--ref <tag\|branch>` | Scaffold a specific stack tag/branch instead of the pinned default (dev). |
| `--source <path>` | Copy a local stack checkout instead of downloading (dev). |

After scaffolding, extend the stack with your agent — see the **`chasqui-create-channel`**
skill for adding a channel, and `chasqui generate module` below for tool modules.

## `chasqui generate module <name>` — scaffold a Tool Module

```bash
chasqui generate module orders --with-models --with-admin
```

Scaffolds a Tool Module (ARCHITECTURE §8) inside an existing project (`snake_case`
name). Options:

- `--with-models` — add a SQLModel table registered via `register_models()`
  (then `cd core && make makemigrations m="add orders tables" && make migrate`).
- `--with-admin` — add admin routes under `/admin/modules/<name>`.

The module contract (what a Tool Module must implement) is in
ARCHITECTURE §8.2 — read it before filling in the real logic.

## Conventions to respect

- **Don't unpin casually.** `--ref main` scaffolds unreleased code; the default
  pinned tag is the supported combination.
- **Secrets live in `.env`** (gitignored), never committed. The wizard writes
  them per service.
- **English-only codebase**; user-facing literals are localized via `.env` and the
  DB system prompt, not hardcoded.
