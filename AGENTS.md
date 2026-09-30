---
tags:
  - dsh
  - plugin
  - opencode
  - cordis
status: note
aliases:
  - dsh-opencode
  - dsh-opencode-session
---

# `dsh-opencode`

> [!info] Summary DeepSeek Harness host plugin that manages session affinity, OpenCode Zen gateway origin headers, and free-tier compatibility for OpenCode, OpenCode Go, and OpenCode Zen routes. Standards reference: [[OBSIDIAN]] (`~/dev/OBSIDIAN.md`).

## Responsibilities

1. **Session Affinity & Caching**: Intercepts `llm/stream` events on OpenCode providers and derives a deterministic `ses_<12hex><14base62>` identifier from DSH's internal conversation UUIDs (`openCodeSessionIdFor`). This keeps prompt caching sticky and prevents `400 MissingSessionID` errors on OpenCode gateways.
2. **Gateway Verification**: DeepSeek Harness's `dsh-llm-pi-ai` strips `User-Agent` from custom provider headers by default. `patchFetch` restores `User-Agent: opencode/1.18.33 ...`, `x-opencode-client: cli`, and `x-opencode-project: global` on outgoing calls to `https://opencode.ai/zen/` so that free-tier origin validation passes.
3. **Core Tool Fallback**: Automatically guarantees that free-tier `/responses` calls carry the expected `read` (`filePath`) and `bash` (`command`) tool definitions, avoiding `403 FreeTierError` on tool-less queries or subagent invocations.

## Repository & Linking

- Source: `~/dev/dsh-opencode`
- Linked in DSH Web profile: `~/.dsh/profiles/web/package.json` as `"dsh-opencode": "link:../../../dev/dsh-opencode"`
- Patch file: `cordis.patch.yml`

## Commands

```sh
pnpm install     # install dependencies
pnpm run build   # vp pack -> lib/index.mjs + lib/index.d.mts
pnpm run check   # vp check (format + lint + types)
pnpm run test    # vp test (vitest)
pnpm typecheck   # tsc --noEmit
```
