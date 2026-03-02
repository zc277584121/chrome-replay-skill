# Chrome DevTools MCP — Setup Context

This file contains background knowledge for the replay skill. The skill assumes MCP is already working.

---

## MCP Configuration

### Cursor (`~/.cursor/mcp.json`)

```json
"Chrome DevTools": {
  "command": "npx",
  "args": ["chrome-devtools-mcp@latest", "--autoConnect"]
}
```

### Claude Code

```bash
claude mcp add chrome-devtools --scope user npx chrome-devtools-mcp@latest
```

## How to Start Chrome for MCP

1. Open Chrome **normally** (no command-line flags needed)
2. Go to `chrome://inspect/#remote-debugging`
3. Enable **"Allow remote debugging for this browser instance"**
4. Wait until status shows **"Server running at: 127.0.0.1:9222"**

## Known Issues

### `Network.enable timed out`
**Cause**: Chrome suspends inactive tabs. MCP times out trying to attach to them.  
**Fix**: Click through all open tabs to fully load them, then retry immediately.  
**Root cause**: Chrome 145 bug — fixed in Chrome 146+.

### `DevTools remote debugging requires a non-default data directory`
**Cause**: Chrome 136+ blocks `--remote-debugging-port` with the default profile directory.  
**Fix**: Use the UI toggle in `chrome://inspect/#remote-debugging` instead of command-line flags. If you must use CLI, use a temp dir: `--user-data-dir=/tmp/chrome-debug`.

### Chrome stuck on "starting..."
**Cause**: A previous Chrome instance launched with `--remote-debugging-port` is still running and conflicts with the UI toggle.  
**Fix**: Kill all Chrome processes (`pkill -9 -f "Google Chrome"`), reopen Chrome normally, re-enable the toggle.

## Multi-Profile Behavior

- MCP connects to the **default profile** (the one Chrome opens by default).
- `list_pages` returns **all tabs across all windows** in that profile — no per-window isolation.
- Multiple profiles are not supported simultaneously ([issue #694](https://github.com/ChromeDevTools/chrome-devtools-mcp/issues/694)).
