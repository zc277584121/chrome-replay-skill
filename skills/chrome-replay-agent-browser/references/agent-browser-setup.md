# agent-browser CLI — Setup Context

This file contains background knowledge for the replay skill. The skill assumes agent-browser is already installed and working.

---

## Installation

```bash
npm install -g agent-browser
```

Or use npx without installing globally:

```bash
npx agent-browser --help
```

Verify installation:

```bash
agent-browser --version
```

## Connecting to an Existing Chrome Instance

agent-browser can connect to a running Chrome with remote debugging enabled. This lets you replay recordings in your real browser with existing cookies, sessions, and login state.

### Option 1: Auto-connect (recommended)

```bash
agent-browser --auto-connect open https://example.com
```

This auto-discovers a Chrome instance with remote debugging enabled.

### Option 2: Explicit CDP port

```bash
agent-browser --cdp 9222 open https://example.com
```

### How to enable remote debugging in Chrome

**Via UI (Chrome 136+, recommended):**
1. Open Chrome normally
2. Go to `chrome://inspect/#remote-debugging`
3. Enable **"Allow remote debugging for this browser instance"**
4. Wait until status shows **"Server running at: 127.0.0.1:9222"**

**Via command line:**
```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome --remote-debugging-port=9222

# Linux
google-chrome --remote-debugging-port=9222

# Windows
"C:\Program Files\Google\Chrome\Application\chrome.exe" --remote-debugging-port=9222
```

> Note: Chrome 136+ blocks `--remote-debugging-port` with the default profile. Use the UI toggle instead, or add `--user-data-dir=/tmp/chrome-debug` when using CLI.

## Launching a Fresh Browser

By default, `agent-browser open <url>` launches a new Chromium instance (Playwright's bundled browser). This is isolated — it does **not** share cookies or sessions with your regular Chrome. Useful for clean-state testing.

## Key Commands Reference

| Command | Description |
|---------|-------------|
| `open <url>` | Navigate to URL |
| `snapshot -i` | List interactive elements with refs (`@e1`, `@e2`, ...) |
| `click @eN` | Click element by ref |
| `fill @eN "text"` | Clear and fill standard input/textarea |
| `type @eN "text"` | Type without clearing |
| `keyboard inserttext "text"` | Insert text (best for contenteditable) |
| `press <key>` | Press keyboard key (Enter, Tab, Escape, etc.) |
| `scroll down/up <amount>` | Scroll page in pixels |
| `wait @eN` | Wait for element to appear |
| `wait --load networkidle` | Wait for network to settle |
| `wait <ms>` | Wait for a duration |
| `screenshot [path]` | Take screenshot |
| `screenshot --annotate` | Screenshot with numbered element labels |
| `eval <js>` | Execute JavaScript in page |
| `get text body` | Get all text content (traverses iframes) |
| `get url` | Get current URL |
| `set viewport <w> <h>` | Set viewport size |
| `find text "..." click` | Semantic find and click |
| `close` | Close browser session |

## Known Issues

### `snapshot -i` returns empty on iframe-heavy pages

**Cause**: agent-browser's snapshot operates on the main frame only. Sites like LinkedIn and Gmail render content inside iframes.
**Workaround**: Use `eval` to access iframe content via `document.querySelector('iframe').contentDocument`, or use `get text body` to read full page text.

### `find text` strict mode error

**Cause**: Multiple elements match the text query; Playwright's strict mode rejects ambiguity.
**Workaround**: Use `snapshot -i` to find the specific ref, or make the text query more specific.

### `fill` does not work on contenteditable

**Cause**: `fill` targets `<input>` and `<textarea>` value properties. Contenteditable divs use a different DOM mechanism.
**Workaround**: Click/focus the element first, then use `keyboard inserttext "text"`.

### Connection refused / timeout

**Cause**: Chrome's remote debugging is not enabled or is using a different port.
**Fix**: Ensure Chrome is running with remote debugging enabled (see "How to enable remote debugging" above). Check that no firewall is blocking port 9222.
