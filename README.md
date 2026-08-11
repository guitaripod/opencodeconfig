# opencodeconfig

Shared opencode configuration for Marcus's machines (Arch desktop + macOS MacBook "mac").

## Layout

- `~/.config/opencode/` is a git clone of this repo on **both** machines — edit here, commit, push, `git pull` on the other.
- Everything except Ollama models is shared: `opencode.json` (providers, MCP servers, permissions), `command/*.md`, `plugin/`, `AGENTS.md`, `tui.json`.
- **Per-machine Ollama models** live in `opencode.local.json` (gitignored, different on each machine) and are merged in via the `OPENCODE_CONFIG` env var exported in `~/.bashrc` (Arch) / `~/.zshrc` (Mac). opencode deep-merges global + custom config, so the local file only contains `provider.ollama.models`.

## Keeping machines in sync

1. Edit a shared file in this repo, commit, push, then `ssh mac 'cd ~/.config/opencode && git pull'`.
2. The Mac's `package-lock.json` drifts (npm regenerates it); if `git pull` complains about unstaged changes there, `git checkout -- package-lock.json` first.
3. SSH alias: `ssh mac` (HostName macbook, user marcus, ed25519 key).

## Adding / changing models

Models are machine-specific — pull a model, then add it to THAT machine's `opencode.local.json` only. Do not add Ollama models to the shared `opencode.json`; the two machines intentionally carry different model sets. New shell sessions need re-sourcing (`source ~/.bashrc` / `~/.zshrc`) to pick up `OPENCODE_CONFIG`.

## Gotchas

- `node_modules`, `package.json`, `bun.lock`, `opencode.local.json`, `.gitignore` are gitignored; plugin files are plain JS/TS with no deps.
- MCP `command` arrays are spawned directly (no shell), so no `$HOME` expansion — use `{env:HOME}` for paths (works config-wide, verified) and bare PATH binaries for executables.
- `flyr` MCP runs `["flyr", "mcp"]` from `~/.cargo/bin` on both machines (`cargo install --path ~/Dev/rust/flyr`); the flyr repo is cloned at `~/Dev/rust/flyr` on both.
- ASC key is at `~/.appstoreconnect/private_keys/AuthKey_DSS2FFU68G.p8` on both machines (`{env:HOME}/.appstoreconnect/...`).
- `kimi-code` provider is in the shared config but the Mac has no `KIMI_CODE_KEY` exported — only usable on Arch until added to `~/.zshrc` there.
- `.claude/workflows/*.js` (flyr.js, hinta-best.js, wwdc.js on the Mac) are Claude Code-only — opencode does not run them; the opencode equivalent is `command/*.md`.
- Small/untrained-for-tools Ollama models (gemma3:1b, tinyllama, smollm2, qwen3:0.6b, gemma3:270m) don't call tools — skills/MCP silently won't fire with them. Tools-capable: nemotron-3.5-lightning:30b-mlx, qwen3:14b, qwen3.5:35b-a3b, gpt-oss:20b, gemma4 variants.
