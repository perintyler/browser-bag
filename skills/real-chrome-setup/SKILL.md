---
name: real-chrome-setup
description: One-time setup to let Barry browse as you using your real Chrome profile. Run this once, then use the `browser-mine` trait in future sessions.
---

# Real Chrome Setup

The `browser-mine` mode connects to your already-running Chrome so Barry can
browse pages you're logged into — same cookies, same sessions, same extensions.
It uses Chrome DevTools Protocol via `--autoConnect`, which requires a one-time
opt-in.

## Requirements

- **Chrome 144+** — check at `chrome://version`. Older versions don't support
  the discovery protocol that `--autoConnect` relies on.
- **Chrome must be running** before the tool connects. It attaches to an
  existing instance; it doesn't launch one.

## Setup (once)

1. Open `chrome://inspect` in Chrome
2. Next to **Discover network targets**, click **Configure…**
3. Add `localhost:9222` to the target list
4. Click **Done**
5. Make sure **Enable port forwarding** is *unchecked* — this is a local
   connection, not a remote one

That's it. Chrome now accepts local DevTools connections on port 9222.

## Verify it works

Start a session with the `browser-mine` trait, then call `take_snapshot`. You
should see your open tabs and their content. If you're logged into a site in
Chrome, the snapshot should show logged-in state.

## Troubleshooting

| Problem | Fix |
|---|---|
| Connection refused | Chrome isn't running, or `localhost:9222` wasn't added in `chrome://inspect` → Configure |
| Attaches but no tabs visible | Another debugger is already attached (DevTools open, another MCP session). Only one debugger can connect at a time |
| Wrong tool names | You're in `browser` mode, not `browser-mine`. Check your session's traits — `browser-mine` tools use `take_snapshot`/`click`/`fill`, not `browser_snapshot`/`browser_click` |
| Stale after Chrome restart | The connection drops when Chrome restarts. Start a new session |

## Security

Enabling `chrome://inspect` on `localhost:9222` exposes your browser to local
debugging. Barry is the intended consumer, but any process on your machine could
connect to the same port. This is the same exposure as opening Chrome DevTools —
no worse, no better. Do not add remote addresses to the target list.
