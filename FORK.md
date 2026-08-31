# Theoman9402/pi-subagents FORK.md - why this fork exists, what it changes

This is a private fork of **upstream [`tintinweb/pi-subagents`](https://github.com/tintinweb/pi-subagents)**, which adds sub-agents and workflow orchestration to [pi](https://github.com/earendil-works/pi-coding-agent).

Locally maintained at: `~/.pi/agent/extensions/pi-subagents/` and pushed to our remote fork's `origin` on user request.
Used as: the pi-subagents instance for this machine's pi setup.

## Remotes

- `origin` - our fork on GitHub: Theoman9402/pi-subagents. Where we push.
- `upstream` - the canonical repo: tintinweb/pi-subagents. Where we fetch from to stay in sync.

## Motivation

Fork so we can make our own changes as we see fit, and stay in sync with `upstream` so we don't fall behind on anything they add - fixes, features, refactors.

- **Make our own changes as we see fit.** We can't rely on tintinweb/subagents to do everything we'd like, so we take matters into our own hands. Where our changes conflict with upstream, ours win.
- **Ship our fixes to our own install immediately**, without waiting for an upstream release - e.g. the `(Tools: …)` suffix bug below.
- **Never fall behind `upstream`** - their fixes, features, and refactors should reach our install too.

We sync everything we didn't deliberately change; where we changed something, ours wins.

## Rules

- **Tests must pass before committing** - `npm run lint && npm run typecheck` at minimum. Note some tests may fail on Windows for environment reasons unrelated to the fork (POSIX path assumptions, `EPERM` in worktree tests).

## Fork-specific Divergences

All changes, new features, removed functionalities or basically anything we diverged into are in this section, in their own subsections. Only completed changes go here.

### `ext:${tool-name}` is not visible in the system prompt's `Agent` Definition

**Problem Statement:** The `Agent` tool in the subagent's system prompt only showed built in tools for matching available tools as defined by the subagent definition file, whereas any extension tool such as `ext:foo/bar` did not show up if it was included in the subagent's definition file.

In upstream, `formatToolsSuffix` (in `src/index.ts`) renders an agent's tool scope for the `(Tools: …)` suffix.

**Before:** it read only `cfg.builtinToolNames`, so `ext:` tools never appeared.

**After:** it appends the declared `extSelectors` (already parsed and stored on the agent config) flat after the built-in list. An agent with `tools: read, edit, write, ext:rpiv-web-tools/web_search, ext:rpiv-web-tools/web_fetch` was advertised as `(Tools: read, edit, write)` - silently omitting the extension tools it could actually call, so the orchestrator would route search/fetch work elsewhere.

**Now:** it renders the full list, and `tools: "*, ext:mcp/search"` renders `(Tools: *, ext:mcp/search)` (was `*`).

**Notes:** Selectors are a **declared-intent** claim: exact for eagerly-registered extensions, a routing hint for lazily-registered ones (MCP servers, context-mode) whose tools resolve only after the description is built. `tools: none` with `isolated` or `extensions: false` still renders `(Tools: none)` - those agents genuinely can call nothing.

## How to reproduce the state

```bash
git remote -v

git diff --stat upstream/master..HEAD

git log --oneline upstream/master..HEAD
```
