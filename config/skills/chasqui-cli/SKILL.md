---
name: chasqui-cli
description: "INVOKE when using the chasqui CLI to scaffold or extend a Chasqui stack: creating a new project (uvx chasqui new), running the setup wizard, pinning a stack version with --ref, adding a channel gateway to an existing project (chasqui add channel), or generating a tool module (chasqui generate module). Covers the commands, their options, and the provisioning flow."
---

# chasqui CLI

The `chasqui` CLI scaffolds and extends a Chasqui stack — *zero to a running AI
chat agent in one command*. It's published to PyPI; run it with `uvx` (no install
needed) or install it. New project creation and code generators à la
`rails generate`.

Authoritative reference (read for the release ceremony, version pinning, and the
generator contract):
→ https://raw.githubusercontent.com/chasqui-stack/cli/main/AGENTS.md
→ Stack architecture (what gets scaffolded): https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.4.0/docs/ARCHITECTURE.md

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

Channels: `whatsapp` is the default dish; `telegram` and `web` are **opt-in**.
`web` is the embeddable site widget (ADR-011) — a Node gateway on `:8002`; the
wizard wires the core's `CHANNEL_WEB_SEND_URL` and the gateway's `.env`
(`WEB_ALLOWED_ORIGINS`, rate limits, `ERROR_REPLY`/`UNSUPPORTED_REPLY`).

Options:

| Option | Effect |
|--------|--------|
| `--defaults` | Skip the wizard — placeholder config, CI-friendly. |
| `--channels whatsapp,telegram,web` | Pick channels non-interactively (overrides the wizard; pairs with `--defaults`). |
| `--skip-provision` | Write files only; no uv/npm/createdb/migrate. |
| `--ref <tag\|branch>` | Scaffold a specific stack tag/branch instead of the pinned default (dev). |
| `--source <path>` | Copy a local stack checkout instead of downloading (dev). |

The wizard also offers optional **Extras** (all `.env`-switchable later): media
storage (S3), handoff notifications (webhook/SMTP), Kamal deploy placeholders,
and **speech-to-text for voice notes** (ADR-010) — when picked it writes the
`STT_*` block to `core/.env` (Groq `whisper-large-v3-turbo` default, native
OGG/Opus; a separate `STT_API_KEY`) so an LLM without native audio can answer
voice notes. Unset = the agent asks the user to type it.

After scaffolding, extend the stack with your agent — see `chasqui add channel`
below for the stack's own gateways, the **`chasqui-create-channel`** skill for
building a custom one, and `chasqui generate module` for tool modules.

## `chasqui add channel <name>` — retrofit a gateway (v0.4.0+)

```bash
cd my-agent && uvx chasqui add channel web   # or: telegram, whatsapp
```

Adds one of the stack's channel gateways to an **existing** generated project
(scaffolded before the channel existed, or skipped in the wizard). Run it from
the project root (where `core/.env` lives). It:

1. Detects the stack tag the project was scaffolded from (README; `--ref`
   overrides) and fetches the gateway dir at that tag.
2. Writes the gateway's `.env`, **reusing the core's `INTERNAL_API_KEY`**
   (read, never regenerated) and asking only that channel's questions
   (port, tokens/origins). `--defaults` skips the questions.
3. Appends `CHANNEL_<CH>_SEND_URL` to `core/.env` (idempotent — refuses to
   duplicate) and provisions (`uv sync` / `npm install`; `--skip-provision`
   to opt out).

It never git-commits — it mutates YOUR repo; review and commit. Restart the
core afterwards so it picks up the new `CHANNEL_<CH>_SEND_URL`.

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
