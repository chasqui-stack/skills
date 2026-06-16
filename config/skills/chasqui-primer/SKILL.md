---
name: chasqui-primer
description: "INVOKE FIRST for any work on a Chasqui stack — building, extending, or operating one. Establishes the omakase philosophy, the three services (channel-agnostic core, operator admin, stateless channel gateways), the canonical message contract, and routes you to the right Chasqui skill (CLI, create-channel, …) for the task at hand. Read this before writing any Chasqui code."
---

# Chasqui — primer & router

Chasqui is a **base development stack for building custom AI agents on messaging
channels** (WhatsApp, Telegram, …). This skill orients you and points you to the
next skill. It is a router, not a manual — the source of truth is the versioned
docs, which you fetch on demand (links below).

## The one rule: read the canonical docs, don't guess

Chasqui's design lives in versioned docs, pinned to the stack release these
skills target (**v0.2.5**). When a task needs a detail, fetch the doc — do not
reproduce it from memory:

- **Architecture & canonical contract** (read this first for anything non-trivial):
  https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.2.5/docs/ARCHITECTURE.md
  (§5 canonical message contract, §5.1 outbound `POST /send`, §8.2 module contract)
- **Decisions (ADRs):**
  https://github.com/chasqui-stack/chasqui/tree/v0.2.5/docs/design

## Philosophy — omakase, the Rails way

Chasqui is **opinionated, conventions over configuration**. Dishes are
substitutable (LLM/embeddings via `.env`) but the menu has an owner: Postgres +
pgvector is identity (ADR-002), not a swappable detail. Non-obvious decisions get
written down as **ADRs in the same PR** — docs-as-code, no wiki. Honor the
conventions; don't invent parallel ones.

## The three services (and the contract between them)

| Service | Stack | Role |
|---------|-------|------|
| **core** | FastAPI + LangGraph + Postgres/pgvector | The heart: ingest, orchestrator, memory, RAG, tools, auth |
| **admin** | React 19 + Vite + Tailwind + shadcn/ui | Operator panel |
| **channel gateways** | FastAPI + platform SDK (PyWa, PTB) | Stateless adapters (WhatsApp, Telegram) |

The golden rule: **services talk only through the canonical message contract.
The core never knows a channel exists.** A gateway turns platform events into a
canonical `POST /ingest` and sends replies via `POST /send`. Keep business logic
in `core/` only.

## Which skill next?

Evaluate in order, stop at the first match:

1. **Scaffolding a new project or running the CLI** (`chasqui new`, the wizard) →
   load **`chasqui-cli`**.
2. **Adding a new channel** (a messaging platform that doesn't exist yet) →
   load **`chasqui-create-channel`**.
3. **Giving the agent a new capability / tool** (a Tool Module) →
   load **`chasqui-create-module`**.
4. **Making a non-obvious architectural decision** (record it) →
   load **`chasqui-write-adr`**.
5. **Deploying or operating the stack on a server** (Kamal) →
   load **`chasqui-deploy`**.

If you're unsure what Chasqui is or how the pieces fit, the architecture doc is
always the right next read.
