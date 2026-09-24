---
name: chasqui-create-channel
description: "INVOKE when adding a NEW channel gateway to a Chasqui stack — i.e. supporting a messaging platform that doesn't exist yet (Discord, Instagram, SMS, …). Walks the canonical-contract path: inbound platform events → POST /ingest, replies → POST /send, with the core kept channel-agnostic. Covers the two gateway archetypes — the Python/FastAPI gateway (WhatsApp, Telegram) and the Node monolith (web, when the channel ships its own client) — with the existing gateways as worked examples."
---

# Create a new Chasqui channel

A channel gateway is a **stateless adapter** that translates between a
messaging platform and Chasqui's **canonical message contract**. The whole point
of the architecture: **the core never learns the channel exists** — it only ever
sees canonical messages. Adding a channel is therefore *almost entirely* gateway
work plus one line of core config.

> Read these before writing anything — they are the source of truth, pinned to
> this stack release (v0.5.1):
> - **Canonical contract** (inbound `/ingest` shape, media as base64 data URIs):
>   https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.5.1/docs/ARCHITECTURE.md  (§5)
> - **Outbound seam** (`POST /send`, conversation mode / human handoff):
>   §5.1 of the same doc + ADR-004:
>   https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.5.1/docs/design/adr-004-conversation-mode-outbound-send.md
> - **Worked example decision** (Telegram, library + webhook integration):
>   https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.5.1/docs/design/adr-006-telegram-channel.md
> - **Second archetype decision** (web: Node monolith, anonymous identity, SSE,
>   rehydration) — ADR-011:
>   https://raw.githubusercontent.com/chasqui-stack/chasqui/v0.5.1/docs/design/adr-011-web-channel.md

## Two archetypes — pick one first

The contract is language-agnostic (the core speaks HTTP + JSON and never learns
the gateway's language), so there are **two** gateway archetypes:

1. **Python/FastAPI gateway** — the default, for platforms with their own client
   app and a push API. The Telegram gateway (sprint 9) was ~40% copied from
   WhatsApp — that overlap *is* the channel pattern. Study one as your template:
   - Telegram: https://github.com/chasqui-stack/telegram (tag `v0.5.1`) — `app/main.py`,
     `app/handlers/`, `app/services/`, `app/core/`.
   - WhatsApp: https://github.com/chasqui-stack/whatsapp (tag `v0.5.1`).
2. **Node monolith** (ADR-011) — when the channel **ships its own client** (no
   third-party app to ride on). One repo, one `npm` toolchain: an Express server
   (relays `/ingest`, receives `/send`, holds the live connection to the client)
   plus the Vite/Preact client it serves. Reference: the **web channel**,
   https://github.com/chasqui-stack/web — an embeddable chat bubble for anonymous
   visitors. Its identity is a UUID in `localStorage` sent as
   `contact.external_id`; live outbound is **SSE** (the gateway maps
   visitor → stream; the core's `POST /send` pushes **complete messages**, not
   token chunks). Trust boundary: the browser talks only to the gateway's public
   routes (origin allowlist + rate limit) — the `INTERNAL_API_KEY` never reaches
   the browser.

## Procedure

1. **Read the contract first** (§5 + §5.1 above). Know the exact `/ingest` payload
   and the `/send` request the core will call. Do not guess the shape.

2. **New repo / service, structural sibling of `telegram/`** (or of `web/` for
   the Node archetype). A stateless adapter. Mirror its layout: route/webhook →
   handlers → a service that builds the canonical message → `POST /ingest` on
   the core.

3. **Inbound → `POST /ingest`.** Translate platform events into the canonical
   message: text, and media **inlined as base64 `data:` URIs** (per §5). Ack fast
   — under inbound coalescing (ADR-008) the core may reply with an empty body and
   send the actual reply later via `/send`, so don't expect the reply inline.

4. **Outbound: expose `POST /send`** and call the platform's send API (ADR-004).
   This is the *same* seam the human-handoff and the deferred/coalesced reply use.
   The core dispatches replies here. **Presentation is gateway-local** (ADR-007):
   render the agent's text for your platform — Telegram converts to MarkdownV2;
   the web widget renders the WhatsApp-flavored markup (`*bold*`, `* ` bullets, …)
   in the browser.

5. **Rehydration (only if your client can miss replies).** The core exposes
   **one** generic internal read —
   `GET /conversations/{channel}/{external_id}/messages` (`INTERNAL_API_KEY`),
   the read-mirror of `/ingest` (ARCHITECTURE §5.4, ADR-011). **Any** channel
   whose client reconnects (like the web widget after a closed tab) uses it to
   replay history — never store transcripts in the gateway.

6. **Wire the core (one line).** The core only needs `CHANNEL_<CH>_SEND_URL` set so
   it knows where to send. That's the *only* core change — no business logic, no
   channel awareness. (Confirm against the architecture doc's channel send-url
   config.)

7. **Gateway-local fallback strings.** `ERROR_REPLY` / `UNSUPPORTED_REPLY` live in
   the gateway's `.env` (English by default, set per-deployment in the user's
   language) — they must fire exactly when the core is unreachable, so they cannot
   live in the core.

8. **Tests + e2e.** Mirror the gateway's test suite; run a live round-trip
   (inbound → coherent reply) as the acceptance check.

9. **CLI + docs.** To make it selectable in `chasqui new`, add it to the CLI's
   channel list (see the `chasqui-cli` skill), and add a `docs/<CHANNEL>-SETUP.md`
   mirroring `TELEGRAM-SETUP.md`. Capture any non-obvious decision as an ADR
   (docs-as-code).

## Anti-patterns

- ❌ Putting any business logic in the gateway — it's a pure translator; logic
  lives in `core/`.
- ❌ Teaching the core about the channel beyond `CHANNEL_<CH>_SEND_URL`.
- ❌ Expecting the agent reply in the `/ingest` response — with coalescing it
  arrives later via `/send`.
- ❌ Hardcoding user-facing strings — localize via `.env` (English default).
- ❌ Streaming token chunks through the contract — the core is one-shot for
  *every* channel; a typewriter effect is a gateway-local extension (ADR-011).
