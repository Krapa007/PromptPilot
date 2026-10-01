# PromptPilot — agent instructions

Shared by every coding agent (Claude Code, Cursor, others). Claude Code reads this file through the `@AGENTS.md` import at the top of `CLAUDE.md`. **Edit shared rules here, not in `CLAUDE.md`** — `CLAUDE.md` holds only Claude-specific additions. Commit this file together with `CLAUDE.md`, because `CLAUDE.md` imports it.

## Tools

The repo is indexed by three MCP servers: gitnexus (impact / change analysis), codebase-memory-mcp "CBM" (symbols, callers, architecture) and lean-ctx (compressed file read, search, shell). Prefer them over built-in read/search/shell for code exploration. Fall back to built-ins when an MCP call errors or returns empty, and say so.
- In Cursor, call MCP tools with `CallMcpTool` (schemas under `mcps/<server>/tools/`).
- gitnexus repo: `PromptPilot`. Pass `repo: "PromptPilot"` on every gitnexus call — several repos are indexed on this machine.
- CBM project: path-mangled from the repo path — get the exact name from `list_projects`.
- Reindex gitnexus with `npx gitnexus analyze --index-only`. `--index-only` stops gitnexus from rewriting `AGENTS.md`, `CLAUDE.md` and `.claude/skills/`. Don't run it while a gitnexus MCP call is in flight.

## gitnexus rules

- Run `impact({target, direction: "upstream", repo: "PromptPilot"})` before you edit a symbol other files use (exported/public), a shared component or helper, or a change that spans 3+ files. Report callers and risk. Warn on HIGH/CRITICAL before editing, and don't use `riskSharedAxes` to waive it.
- `risk: UNKNOWN` or an empty caller set is not "safe". Callers can be unresolvable (dynamic dispatch, reflection/DI, plain-object property access). Confirm with a text search before you change or delete the symbol.
- Before a commit: `detect_changes({scope: "all"})`. `partial: true` or `truncated: true` is not a clean result — re-run it.
- Rename symbols with gitnexus `rename` (run it with `dry_run: true` first), not find-and-replace.
- Impact scored against a stale index is worse than none. If the index is behind HEAD, reindex first.
- Concepts and flows: `query`. One named symbol: `context`. Overview: `gitnexus://repo/PromptPilot/context` (also `/clusters`, `/processes`).
