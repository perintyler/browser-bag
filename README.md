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

## Why an upstream server instead of our own tools

This replaces a homegrown `playwright` pack that drove pages by CSS selector
against 1000 characters of body text. `@playwright/mcp` uses accessibility
snapshots with stable element refs: more reliable, and cheaper in tokens.

The version is pinned. The pack this replaced used `@latest`, so an upstream
release could break browsing with no diff on our side.

`skills/web-browsing/` documents the tool names, each transcribed from a live
`tools/list` — the check the previous pack's skill skipped, which is why every
call it taught failed.
