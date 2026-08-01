# browser

Web browsing for Barry, via the official
[`@playwright/mcp`](https://github.com/microsoft/playwright-mcp) server.

    barry start --traits browser

Headless Chromium on a fresh profile — no cookies, no logins. To browse as the
user with their real logged-in Chrome, use the **`browser-mine`** pack instead.

## One Playwright server, deliberately

Session tool filtering resolves traits to a set of bare tool **names**, and
every `@playwright/mcp` instance exports the same 24 `browser_*` names. A
second Playwright server — in this pack or a sibling — therefore survives a
trait filter meant for this one: a session traited for headless was verified to
receive 48 tools from both namespaces, with no way for the model to tell which
browser it was driving.

So headless-vs-headed is a per-run flag on this one server, not a second
server. `browser-mine` can coexist because its vocabulary is entirely
different.

## Access level: `enabled`, not `deferred`

The 24 `browser_*` tools cost ~4.2k tokens, about 27% of a session's tool list.
That looks like a case for `deferred` (hidden from `tools/list`, still callable
via `tool_search`) — but it isn't. Only a session that explicitly asked for the
`browser` trait pays that cost, so deferring would hide the tools from the one
session that requested them and buy a `tool_search` round trip for nothing.

Deferral is for packs that load into every session regardless of intent. Trait
gating already solves the same problem here, earlier.

## Known limitation: the browser is shared across sessions

Barry connects a pack once and pools that connection, so every session holding
a browsing trait shares one browser context and one tab. Verified: session A
set `window.__barry_marker` and navigated; session B read back both.

`--shared-browser-context` is therefore not worth enabling — contexts are
already shared. The inverse (a context per session) is not reachable by a
server flag either: `@playwright/mcp` isolates per client connection, and
Barry presents itself as a single client. Fixing it would mean per-session pack
connections in `PackConnectionPool`, which is a Barry-side change.

## Why an upstream server instead of our own tools

This replaces a homegrown `playwright` pack that drove pages by CSS selector
against 1000 characters of body text. `@playwright/mcp` uses accessibility
snapshots with stable element refs: more reliable, and cheaper in tokens.

The version is pinned. The pack this replaced used `@latest`, so an upstream
release could break browsing with no diff on our side.

`skills/web-browsing/` documents the tool names, each transcribed from a live
`tools/list` — the check the previous pack's skill skipped, which is why every
call it taught failed.
