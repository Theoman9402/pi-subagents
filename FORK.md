# FORK.md - why this fork exists, what it changes

This fork tracks **upstream [`tintinweb/pi-subagents`](https://github.com/tintinweb/pi-subagents)**, the pi extension that adds sub-agents and workflow orchestration to [pi](https://github.com/earendil-works/pi-coding-agent). It is maintained locally at `~/.pi/agent/extensions/pi-subagents/` and used as the pi-subagents instance for this machine's pi setup.

**Motivation.** Upstream is a moving target. We run pi on Windows with a specific set of extensions and a specific way of using sub-agents; upstream evolves independently. Forking lets us:
- **Ship fixes to our own install immediately**, without waiting for an upstream release - e.g. the `(Tools: …)` suffix bug below.
- **Keep our install stable** while upstream churns. We pin to a known-good point and apply only what we need.
- **Contribute back cleanly.** The fork is a staging area: a fix proven here can be PR'd upstream without local hacks leaking into the diff.

**How to stay current.** `git fetch upstream && git merge upstream/master` (or rebase). The fork diverges from upstream only where we deliberately changed something; everything else tracks.

---

## What we changed (the fork delta)

The fork's `master` is upstream plus a small, deliberate delta. The delta below is measured against upstream at commit `4f572ea` (v0.19.0).

| Area | Change |
|---|---|
| `src/index.ts` | `formatToolsSuffix()` now lists declared `ext:` selectors in the `(Tools: …)` suffix, not just built-ins |
| `test/tool-description-mode.test.ts` | New tests lock the suffix behavior |
| `README.md`, `CHANGELOG.md` | Documentation of the above |

**The fix in detail.** `formatToolsSuffix` (in `src/index.ts`) renders an agent's tool scope for the `(Tools: …)` suffix. Before: it read only `cfg.builtinToolNames`, so `ext:` selectors never appeared. After: it appends the declared `extSelectors` (already parsed and stored on the agent config) flat after the built-in list. An agent with `tools: read, edit, write, ext:rpiv-web-tools/web_search, ext:rpiv-web-tools/web_fetch` was advertised as `(Tools: read, edit, write)` - silently omitting the extension tools it could actually call, so the orchestrator would route search/fetch work elsewhere. Now it renders the full list, and `tools: "*, ext:mcp/search"` renders `(Tools: *, ext:mcp/search)` (was `*`).

Selectors are a **declared-intent** claim: exact for eagerly-registered extensions, a routing hint for lazily-registered ones (MCP servers, context-mode) whose tools resolve only after the description is built. `tools: none` with `isolated` or `extensions: false` still renders `(Tools: none)` - those agents genuinely can call nothing.

## How to reproduce the state

```bash
# remotes (already configured on this machine)
git remote -v
#   origin   https://github.com/Theoman9402/pi-subagents.git   (this fork)
#   upstream https://github.com/tintinweb/pi-subagents.git      (upstream)

# the delta vs upstream
git diff --stat upstream/master..HEAD

# the commits unique to this fork
git log --oneline upstream/master..HEAD
```

## Notes for future edits

- **This fork is meant to stay upstreamable.** Changes should be generic - no hardcoded paths, extension names, or machine-specific settings. A fix that only works on this machine is a local hack, not a fork improvement.
- **`master` is the only branch** on the fork; the deleted branches live on upstream, so nothing was lost.
- **Local-only files are untracked** (e.g. anything in `.pi/`), so the fork diff stays clean for PRs.
- **Tests must pass before committing** - `npm run lint && npm run typecheck && npx vitest run test/tool-description-mode.test.ts`. Note some tests fail on Windows for environment reasons unrelated to the fork (POSIX path assumptions, `EPERM` in worktree tests); the failing-file set should stay identical to upstream's.
