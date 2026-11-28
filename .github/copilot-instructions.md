# Copilot instructions

`dsh-layerfs` is the LayerFS UI: a plugin bundle for DeepSeek Harness (dsh) web. It adds the Work and History tabs and the LayerFS page, and changes no dsh code. [AGENTS.md](../AGENTS.md) holds the binding rules and [README.md](../README.md) the install and develop guide; read both before non-trivial work.

## Commands

- Install: `pnpm install` (its `preinstall` links `.layerfs`).
- All tests: `pnpm test`. It stubs LayerFS (fake Docker, recorded traces), starts no container or daemon, and runs in under 10 seconds.
- One test: run its file with node, e.g. `node packages/model/tests/cli.mjs` or `node tools/checks/tests/checks.mjs`. Test files are listed in the `test` script of `package.json`; `test:scenarios` and `test:windows` are separate scripts.
- Build client bundle: `pnpm run build` (needs `DSH_SOURCE`, a built deepseek-harness checkout).
- Before live testing: `pnpm run check:layerfs` fails unless the LayerFS checkout is at the commit pinned in `layerfs.json`.
- No linter or type checker is configured.
- **Every test command needs a wall timeout of at most 120 s.** Never background or loop a test; a test that hits the limit is a hang, so diagnose it from source first. macOS has no `timeout`: use `perl -e 'alarm 120; exec @ARGV' <command>`.

## Architecture

- `packages/host`: the `/layerfs` routes (`src/route.js`), UI reads (`trees.js`, `reads.js`, `storage.js`) and the agent wrappers (`src/agent/sdk.js`, `backend.js`, `cli.js`).
- `packages/model` is the view model, `packages/view` the views, `packages/client` the browser client (bundled by `tools/build-client.mjs`).
- `packages/mock` is the scripted model, driven by `scenarios/`. Scenarios touch only projects named `mock-…`, every one runs on real LayerFS, and the simulator is gone.
- `packages/dsh-layerfs` is the bundle that lists the plugin rows.
- LayerFS is an external checkout, reached only through the git-ignored `.layerfs` link (set by `LAYERFS_SOURCE`, default `../layerfs`).

## Conventions

- Reach LayerFS only through `.layerfs` or the `@layerfs/agent`, `@layerfs/backend` and `@layerfs/runtime` packages. Never use relative paths into a sibling checkout, deep-import package files, or import `@layerfs/agent/internal`.
- Use the public entry points: `LayerFS.open` / `LayerFS.init` for clients, `LayerFSBackend.of(client)` for backend calls, and the Host's one `LayerFSRuntime` for the worker, Init, removal, status and leftovers.
- Never edit, commit or branch in the LayerFS checkout. If a change needs LayerFS to change, report it for the layerfs repository.
- Agents see `DSH_LAYERFS_*` env names only; `agent/env.js` maps them to `LAYERFS_*`. Any new place an agent's process loads LayerFS must go through it.
- One attempt per call: no automatic retry, busy loop or replay. A lost reply is `Unknown`; show it and never resend a mutation.
- Do not build standalone HTML or TUI pages, and do not add back the retired `dsh-layerfs-demo` bundle.
- Never restart dsh web while a run is live, and never start, stop or restart one you did not start.
- When refactoring, delete a test that is hard to adapt rather than nursing it, and list deleted or skipped tests in the commit message. Never weaken a surviving assertion to make it pass.
