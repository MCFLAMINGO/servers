# AGENTS.md

Repository-wide guidance for coding agents. See `CLAUDE.md` for the full build/test/style reference and per-server `AGENTS.md` files (e.g. `src/everything/AGENTS.md`) for server-specific notes.

## Cursor Cloud specific instructions

This is an npm-workspaces monorepo of 7 standalone MCP reference servers under `src/`: 4 TypeScript (`everything`, `filesystem`, `memory`, `sequentialthinking`) and 3 Python (`fetch`, `git`, `time`). The servers are independent processes with **no shared backend, database, or inter-server dependencies**. Each communicates over **stdio (JSON-RPC) by default**; only `everything` can optionally serve HTTP.

The update script already installs `uv` (to `~/.local/bin`), runs `npm install`, and `uv sync` for each Python server. Standard build/test/lint commands live in `CLAUDE.md` and each `package.json`/`pyproject.toml`; the notes below cover only non-obvious caveats.

### Non-obvious caveats

- **`uv` is not on the default `PATH`** in non-login shells. It installs to `~/.local/bin`. Invoke it as `~/.local/bin/uv` (or add that dir to `PATH`) if `uv` is not found.
- **`npm install` builds all TypeScript servers automatically** via each package's `prepare` script — there is no separate build step needed to produce `dist/`. Re-run `npm install` (or `npm run build`) after editing TS source.
- **The 4 TS servers have no root test script.** Run tests per package: `cd src/<server> && npm test` (vitest). Python: `cd src/<server> && ~/.local/bin/uv run pytest`.
- **Known pre-existing failing test (not env-related):** `src/memory` — `__tests__/file-path.test.ts > should use new file when both old and new files exist` fails on a clean checkout because `ensureMemoryFilePath()` migrates/removes the legacy `memory.json` even when the new `memory.jsonl` already exists. All other TS/Python tests pass.
- **`fetch` requires outbound network**; `git` needs a repo path (e.g. `--repository /workspace`); `filesystem` requires at least one allowed directory arg; `memory` persists to `MEMORY_FILE_PATH` (defaults to `memory.jsonl` next to the module).

### Running / testing a server end-to-end

Servers idle on stdio waiting for a client. Drive them with the MCP Inspector or raw JSON-RPC:

- Inspector (web UI): `npx -y @modelcontextprotocol/inspector` (add `DANGEROUSLY_OMIT_AUTH=true` to skip the auth token).
- `everything` over HTTP: `cd src/everything && PORT=3001 node dist/index.js streamableHttp`, then connect the Inspector to `http://localhost:3001/mcp` with transport "Streamable HTTP".
- TS via Inspector CLI: `npx @modelcontextprotocol/inspector node src/<server>/dist/index.js [args]`
- Python via Inspector CLI: `npx @modelcontextprotocol/inspector ~/.local/bin/uv --directory src/<server> run mcp-server-<name> [args]`
