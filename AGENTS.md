---
tags:
  - dsh
  - plugin
  - opencode
  - cordis
status: note
aliases:
  - dsh-opencode
---

# `dsh-opencode` — OpenCode on DeepSeek Harness

> [!info] Summary DSH host plugin (`@viztor/dsh-opencode` on npm, repo `viztor/dsh-opencode`) that keeps OpenCode Zen free-tier models working inside DeepSeek Harness: deterministic `ses_…` session affinity, gateway origin-header restoration, and `read`/`bash` tool-schema fallback. Standards reference: [[OBSIDIAN]] (`~/dev/OBSIDIAN.md`).

## How it works

1. **Turn scope** — `apply()` hooks `llm/stream` for configured providers, derives a stable `ses_<12hex><14base62>` ID per DSH session (`openCodeSessionIdFor`, SHA-256), and carries it in `AsyncLocalStorage` across the streamed turn (`withStore`).
2. **Fetch patch** — `patchFetch()` intercepts only OpenCode traffic (`isOpenCodeRequest`: `opencode.ai/zen` URL or matching provider in turn state). It always sets `x-opencode-session`, optionally restores `User-Agent` / `x-opencode-client` / `x-opencode-project`, and injects fallback `read`+`bash` schemas into free-tier `/responses` bodies. Non-OpenCode requests return via the original fetch untouched.
3. **Settings UI** — `src/settings-page.tsx` builds `lib/client.js`, contributing the OpenCode Integration card under DSH Settings → Plugins (toggles for every injection + provider list + UA override).

## Repo map

- `src/index.ts` — host plugin: config, session hashing, ALS store, fetch patch, `apply`. No `as`, arrow consts, sync Promise wrappers (ALS-safe by design).
- `src/settings-page.tsx` — Web client bundle (React, en/zh). Guard unknown scope with `isSettingsFormScope`, never assert.
- `test/plugin.test.ts` — 44 deterministic Vitest cases; polling helper instead of sleeps; restores `globalThis.fetch`/env.
- `scripts/name-client-bundle.ts` — renames `vp pack`'s `.cjs` output to `lib/client.js` (DSH loader requires `.js`).
- `cordis.patch.yml` — default plugin row (`id: dsh-opencode`); header comments are the headless-config reference.
- `.github/workflows/` — `ci.yml` (push/PR: check+test+build), `release.yml` (tag `v*.*.*`: verify, guard tag==version, OIDC `npm publish`).
- `README.md` consumer docs · `CONTRIBUTING.md` dev conventions + release · `CHANGELOG.md` per-version record.

## Commands & policies

```sh
pnpm install     # install dependencies
pnpm run build   # vp pack -> lib/index.mjs + lib/index.d.mts + lib/client.js
pnpm run check   # zero warnings/errors required
pnpm run test    # 44 deterministic tests, fully green required
```

- Release: bump `package.json` + `CHANGELOG.md`, commit, `git tag vX.Y.Z && git push origin vX.Y.Z` (OIDC publishes; no tokens).
- DSH Web profile wires the published build: `~/.dsh/profiles/web/package.json` deps + `bundles` use `@viztor/dsh-opencode` (`link:` only for local dev).
- Hygiene: never hardcode `ses_…`/keys in src/tests/git; `lib/` gitignored; `OPENCODE_SESSION_ID` env override only.
