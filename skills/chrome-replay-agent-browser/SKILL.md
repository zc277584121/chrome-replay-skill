---
name: chrome-replay-agent-browser
description: Replay Chrome DevTools Recorder exports in a live browser session using agent-browser CLI (Playwright-based). Handles JSON, Puppeteer JS, and @puppeteer/replay JS formats with semantic selector matching.
---

# Skill: Replay Chrome DevTools Recording (agent-browser)

Replay a Chrome DevTools Recorder export in the user's live browser via the [agent-browser](https://github.com/vercel-labs/agent-browser) CLI.

> **Prerequisite**: agent-browser must be installed and available. See `references/agent-browser-setup.md` if unsure.

---

## Accepted Input Formats

The user may provide any of these exports from Chrome DevTools Recorder:

**1. JSON** (recommended)

Preferred over JS formats for two reasons:
- Structured and unambiguous to parse
- **Large recordings can be read progressively** instead of loading the entire file into context at once. Use `jq` or Python to inspect specific steps:

```bash
# count steps
jq '.steps | length' recording.json

# read steps 0–4 only
jq '.steps[0:5]' recording.json

# find all navigate steps
jq '[.steps[] | select(.type == "navigate")]' recording.json
```

```python
import json
with open("recording.json") as f:
    steps = json.load(f)["steps"]
# process steps[i] one at a time
```

Start by reading just the first few steps to understand intent, then read further as needed.

```json
{
  "title": "My recording",
  "steps": [
    { "type": "navigate", "url": "https://example.com" },
    { "type": "click", "selectors": [["aria/Submit"], ["button.submit"]] },
    { "type": "change", "value": "hello", "selectors": [["aria/Search box"]] }
  ]
}
```

**2. @puppeteer/replay JS** (`import { createRunner }`)
**3. Puppeteer JS** (`require('puppeteer')`, `page.goto`, `Locator.race`)

All three carry the same semantic information. Parse whichever is provided.

---

## How to Replay

**Read semantically, not literally.** The recording is a reference — selectors, IDs, and coordinates may be stale. The page state may differ (popups, login walls, different content).

### Step-by-step approach

1. **Parse the recording** — understand the full intent before acting. Summarize what the recording does in 1–2 sentences.

2. **Navigate first** — execute `navigate` steps directly.

3. **For each interaction step**, take a snapshot first, then find the target element:
   - Run `agent-browser snapshot -i` to get interactive elements with refs (`@e1`, `@e2`, ...)
   - Match the recording's `aria/...` selectors against the snapshot output
   - Fall back to `text/...`, then CSS class hints, then visual context from a screenshot
   - **Do not rely on ember IDs, numeric IDs, or exact XPaths** — these change every page load
   - If the snapshot returns empty or is missing expected elements, you may have hit an iframe page — see [Iframe-Heavy Sites](#iframe-heavy-sites) below

4. **Step type mapping**:
   | Recording type | agent-browser action |
   |---|---|
   | `navigate` | `agent-browser open <url>` then `agent-browser wait --load networkidle` |
   | `click` | `agent-browser snapshot -i` → find ref → `agent-browser click @eN` |
   | `change` (standard input) | `agent-browser click @eN` → `agent-browser fill @eN "text"` |
   | `change` (contenteditable) | `agent-browser click @eN` → `agent-browser keyboard inserttext "text"` |
   | `keyDown` / `keyUp` | `agent-browser press <key>` |
   | `scroll` | `agent-browser scroll down <amount>` or `scroll up <amount>` |
   | `setViewport` | `agent-browser set viewport <width> <height>` |
   | `waitForElement` | `agent-browser wait @eN` or `agent-browser wait "<css-selector>"` |

   **How to distinguish `change` targets**: If the snapshot shows the element as `textbox`, `input`, or `textarea`, use `fill`. If it shows `[contenteditable]`, `div[role="textbox"]`, or is a rich text editor (LinkedIn message box, Gmail compose, Slack, Notion), use `keyboard inserttext`.

5. **After each significant step**, take a snapshot (`agent-browser snapshot -i`) or screenshot (`agent-browser screenshot`) to confirm the result before proceeding.

6. **Ref lifecycle**: Refs (`@e1`, `@e2`, ...) are invalidated when the page changes. Always re-snapshot after clicking links, submitting forms, or triggering dynamic content loads.

---

## Iframe-Heavy Sites

agent-browser's `snapshot -i` operates on the main frame only and **cannot penetrate iframes**. Sites like LinkedIn, Gmail, and embedded editors often render key interactive content inside iframes.

### Detecting iframe issues

- `snapshot -i` returns an unexpectedly short or empty list for a page that visually has many interactive elements
- The recording references elements (buttons, inputs) that do not appear in the snapshot output
- `agent-browser get text body` returns content that doesn't match what you see in a screenshot

### Workarounds

1. **Use `eval` to access iframe content directly**:
   ```bash
   # Find and click a button inside an iframe
   agent-browser eval --stdin <<'EVALEOF'
   const frame = document.querySelector('iframe[data-testid="interop-iframe"]');
   const doc = frame.contentDocument;
   const btn = doc.querySelector('button[aria-label="Send"]');
   btn.click();
   EVALEOF
   ```
   Note: `eval` runs in the main frame. You must traverse into the iframe via `iframe.contentDocument`. This only works for same-origin iframes.

2. **Use `keyboard` for blind input**: If the target element inside the iframe has focus (e.g., after clicking into it via `eval`), `agent-browser keyboard inserttext "..."` sends text to the focused element regardless of frame boundaries.

3. **Use `get text body`** to read full page content (including iframes) when `snapshot` fails. This helps identify available elements and text.

4. **Use `screenshot`** to visually verify the page state when snapshot data is unreliable.

### When to ask the user

If none of the workarounds succeed after 2 attempts on the same step, pause and explain:
- The page uses iframes that agent-browser cannot directly access via snapshot
- Which element you need to interact with and what the recording expects
- Ask the user to perform that specific step manually, then continue with the next step

---

## Handling Unexpected Situations

**Handle these automatically (do not stop):**
- Unexpected popups or banners → dismiss them (`agent-browser find text "Dismiss" click` or `find text "Close" click`), then continue
- Cookie consent dialogs → accept or dismiss
- Tooltip overlays blocking clicks → close them first
- Element not found in snapshot → try `agent-browser find text "..." click` as fallback, or scroll to reveal the element with `agent-browser scroll down 300`

**Pause and ask the user when:**
- Login / authentication is required
- A CAPTCHA appears
- The page structure looks completely different from what the recording implies
- A destructive action is about to happen (e.g., deleting data, submitting a form that sends real content) — confirm with user before proceeding
- You are stuck for more than 2 attempts on the same step
- An iframe-heavy page where all workarounds have failed

When pausing, explain clearly: what step you are on, what you expected, and what you actually see.

---

## Known Limitations

1. **Iframe blindness**: `snapshot -i` cannot see inside iframes. See [Iframe-Heavy Sites](#iframe-heavy-sites) above.

2. **`find text` strict mode**: `agent-browser find text "X"` fails when multiple elements match. Workaround: use `snapshot -i` to locate the specific ref, or make the text query more specific.

3. **`fill` vs contenteditable**: `fill` targets standard `<input>` and `<textarea>` elements. For `contenteditable` divs (rich text editors, social media message boxes), use `keyboard inserttext` instead.

4. **`eval` is main-frame only**: JavaScript executed via `agent-browser eval` runs in the top-level frame. To interact with iframe content, navigate the DOM: `document.querySelector('iframe').contentDocument...`

---

> For installation instructions, connecting to Chrome, and command reference, see [`references/agent-browser-setup.md`](./references/agent-browser-setup.md).
