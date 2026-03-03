# chrome-replay-skill

A collection of skills for AI coding agents to replay [Chrome DevTools Recorder](https://developer.chrome.com/docs/devtools/recorder) exports in a live browser session.

Two backend options are provided — pick the one that fits your setup:

## Available Skills

| Skill | Backend | Best for |
|-------|---------|----------|
| [`chrome-replay-devtools`](./skills/chrome-replay-devtools/) | [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp) | Agents with MCP support (Claude Code, Cursor, etc.) |
| [`chrome-replay-agent-browser`](./skills/chrome-replay-agent-browser/) | [agent-browser CLI](https://github.com/vercel-labs/agent-browser) (Playwright) | Any agent with shell access; no MCP required |

### Which should I use?

- **chrome-replay-devtools** — Replays inside your existing Chrome session via Chrome DevTools Protocol. Best iframe support (CDP accessibility tree penetrates iframes). Requires an MCP-capable agent and Chrome DevTools MCP to be configured.
- **chrome-replay-agent-browser** — Replays via shell commands using the agent-browser CLI (Playwright-based). Works with any agent that can run shell commands. Can connect to existing Chrome (`--auto-connect`) or launch a fresh browser. No MCP configuration needed. Note: limited iframe support — see the skill's [Known Limitations](./skills/chrome-replay-agent-browser/SKILL.md#known-limitations).

## What it does

Both skills share the same core capabilities:

- Accept recordings exported from Chrome DevTools Recorder (JSON, Puppeteer JS, or @puppeteer/replay JS)
- Semantically replay the recorded steps — handle stale selectors, unexpected popups, and page state differences
- Pause and ask for human input when login, CAPTCHAs, or destructive actions are encountered

## How to Record

1. Open the target webpage in Chrome
2. Open DevTools (F12)
3. Click **"More tools"** → **"Recorder"** (or search for "Recorder" in the DevTools command menu)
4. Click **"Create a new recording"**
5. Click **"Start recording"**
6. Perform the actions you want to replay (click, type, navigate, etc.)
7. Click **"End recording"**
8. Export as **JSON** (recommended), Puppeteer JS, or @puppeteer/replay JS

## Installation

Install using [npx skills](https://skills.sh):

### Install both skills

```bash
npx skills add zc277584121/chrome-replay-skill -a <agent-name>

# Global (available in all projects)
npx skills add zc277584121/chrome-replay-skill -a <agent-name> -g
```

### Examples

```bash
# Claude Code
npx skills add zc277584121/chrome-replay-skill -a claude-code -g

# Cursor
npx skills add zc277584121/chrome-replay-skill -a cursor -g

# Codex
npx skills add zc277584121/chrome-replay-skill -a codex -g
```

### Other Agents

`npx skills` supports 40+ agents. Use `-a <agent-name>` to target any supported agent:

```bash
npx skills add zc277584121/chrome-replay-skill -a <agent-name>
```

Common agent names: `windsurf`, `github-copilot`, `cline`, `roo`, `gemini-cli`, `goose`, `kilo`, `augment`, `opencode`.

> **Project vs Global**: Project-level installs the skill into the current project directory (e.g., `.claude/skills/`). Global (`-g`) installs to your home directory (e.g., `~/.claude/skills/`) so it's available across all projects.

## Updating

```bash
# Check if updates are available
npx skills check

# Update all globally installed skills to latest
npx skills update
```

> **Note**: `npx skills update` only works for globally installed skills (`-g`). For project-level installs, re-run the `npx skills add` command to get the latest version.

## Requirements

- **chrome-replay-devtools**: Chrome DevTools MCP must be configured and connected. See [setup guide](./skills/chrome-replay-devtools/references/chrome-devtools-mcp-setup.md) or the [official Chrome DevTools MCP repo](https://github.com/ChromeDevTools/chrome-devtools-mcp).
- **chrome-replay-agent-browser**: agent-browser CLI must be installed (`npm install -g agent-browser`). See [setup guide](./skills/chrome-replay-agent-browser/references/agent-browser-setup.md) or the [agent-browser repo](https://github.com/vercel-labs/agent-browser).
