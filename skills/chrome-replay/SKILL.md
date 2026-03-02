---
name: chrome-replay
description: Replay Chrome DevTools Recorder exports in a live browser session using Chrome DevTools MCP. Handles JSON, Puppeteer JS, and @puppeteer/replay JS formats with semantic selector matching.
---

# Skill: Replay Chrome DevTools Recording

Replay a Chrome DevTools Recorder export in the user's live browser via the Chrome DevTools MCP.

> **Prerequisite**: Chrome MCP must be connected. See `references/chrome-devtools-mcp-setup.md` if unsure.

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
   - Try `aria/...` selectors first (most robust)
   - Fall back to `text/...`, then CSS class hints, then visual context from the snapshot
   - **Do not rely on ember IDs, numeric IDs, or exact XPaths** — these change every page load

4. **Step type mapping**:
   | Recording type | MCP action |
   |---|---|
   | `navigate` | `navigate_page` |
   | `click` | `take_snapshot` → find uid → `click` |
   | `change` | `click` the element → `type_text` or `fill` |
   | `keyDown/keyUp` | `press_key` |
   | `scroll` | `evaluate_script` with `window.scrollBy` |
   | `setViewport` | `emulate` viewport |
   | `waitForElement` | `wait_for` |

5. **After each significant step**, take a snapshot to confirm the result before proceeding.

---

## Handling Unexpected Situations

**Handle these automatically (do not stop):**
- Unexpected popups or banners → dismiss them (`aria/Dismiss`, `aria/Close`, `aria/×`) then continue
- Cookie consent dialogs → accept or dismiss
- Tooltip overlays blocking clicks → close them first

**Pause and ask the user when:**
- Login / authentication is required
- A CAPTCHA appears
- The page structure looks completely different from what the recording implies
- A destructive action is about to happen (e.g., deleting data, submitting a form that sends real content) — confirm with user before proceeding
- You are stuck for more than 2 attempts on the same step

When pausing, explain clearly: what step you are on, what you expected, and what you actually see.

---

> For MCP setup, known issues, and Chrome startup instructions, see [`references/chrome-devtools-mcp-setup.md`](./references/chrome-devtools-mcp-setup.md).
