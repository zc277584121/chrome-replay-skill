# chrome-replay-skill

A skill for AI coding agents to replay [Chrome DevTools Recorder](https://developer.chrome.com/docs/devtools/recorder) exports in a live browser session, using the [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp).

## What it does

- Accepts recordings exported from Chrome DevTools Recorder (JSON, Puppeteer JS, or @puppeteer/replay JS)
- Semantically replays the recorded steps — handles stale selectors, unexpected popups, and page state differences
- Pauses and asks for human input when login, CAPTCHAs, or destructive actions are encountered

## Files

- `SKILL.md` — the skill instructions for the AI agent
- `context.md` — Chrome DevTools MCP setup reference and known issues

## Requirements

Chrome DevTools MCP must be configured and connected. See `context.md` for setup instructions, or refer to the [official Chrome DevTools MCP repository](https://github.com/ChromeDevTools/chrome-devtools-mcp).
