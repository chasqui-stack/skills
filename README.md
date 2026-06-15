# Chasqui Skills

[![skills.sh](https://skills.sh/b/chasqui-stack/skills)](https://skills.sh/chasqui-stack/skills)

[Agent Skills](https://agentskills.io) that teach **any** skills-compatible coding
agent (Claude Code, Cursor, Codex, Gemini CLI, …) how to work with
[Chasqui](https://github.com/chasqui-stack/chasqui) — the base stack for building
custom AI agents on WhatsApp, Telegram, and more.

These skills are **thin pointers** to Chasqui's versioned docs, not copies — they
carry the *procedure* and tell your agent which canonical doc to read on demand,
pinned to the stack release they target. See
[ADR-009](https://github.com/chasqui-stack/chasqui/blob/main/docs/design/adr-009-extension-skills.md)
for the design.

## Skills

| Skill | Use it when |
|-------|-------------|
| **chasqui-primer** | First — orients you (philosophy, the 3 services, the contract) and routes you to the right skill. |
| **chasqui-cli** | Scaffolding a stack with the CLI (`uvx chasqui new`, the wizard, `chasqui generate module`). |
| **chasqui-create-channel** | Adding a new channel gateway (a messaging platform that doesn't exist yet). |
| **chasqui-create-module** | Giving the agent a new capability — a tool / Tool Module (tools, tables, admin config). |
| **chasqui-write-adr** | Recording a non-obvious architectural decision the Chasqui way (docs-as-code ADR). |
| **chasqui-deploy** | Deploying or operating the stack on a server (Kamal 2). |

## Install

### `npx skills` (any Agent-Skills client)

```bash
# this project
npx skills add chasqui-stack/skills --skill '*' --yes

# all projects
npx skills add chasqui-stack/skills --skill '*' --yes --global

# pin to a specific agent (e.g. Claude Code)
npx skills add chasqui-stack/skills --agent claude-code --skill '*' --yes
```

### Claude Code plugin

```bash
/plugin marketplace add chasqui-stack/skills
/plugin install chasqui-skills@chasqui-skills
```

## Versioning

Skills are versioned in **lockstep with the Chasqui stack** (`STACK_TAG`): the
doc pointers inside each skill pin the exact release they target. Install the
skills that match the stack version you're building on.

## License

Apache-2.0
