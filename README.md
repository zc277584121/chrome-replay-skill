# chrome-replay-skill

A skill for AI coding agents to replay [Chrome DevTools Recorder](https://developer.chrome.com/docs/devtools/recorder) exports in a live browser session, using the [Chrome DevTools MCP](https://github.com/ChromeDevTools/chrome-devtools-mcp).

## What it does

- Accepts recordings exported from Chrome DevTools Recorder (JSON, Puppeteer JS, or @puppeteer/replay JS)
- Semantically replays the recorded steps — handles stale selectors, unexpected popups, and page state differences
- Pauses and asks for human input when login, CAPTCHAs, or destructive actions are encountered

## Files

- `SKILL.md` — the skill instructions for the AI agent
- `context.md` — Chrome DevTools MCP setup reference and known issues

## Installation

Install this skill using [npx skills](https://skills.sh):

### Claude Code

```bash
# Project-level (current project only)
npx skills add zc277584121/chrome-replay-skill -a claude-code

# Global (available in all projects)
npx skills add zc277584121/chrome-replay-skill -a claude-code -g
```

### Cursor

```bash
# Project-level
npx skills add zc277584121/chrome-replay-skill -a cursor

# Global
npx skills add zc277584121/chrome-replay-skill -a cursor -g
```

### Codex

```bash
# Project-level
npx skills add zc277584121/chrome-replay-skill -a codex

# Global
npx skills add zc277584121/chrome-replay-skill -a codex -g
```

### OpenCode

```bash
# Project-level
npx skills add zc277584121/chrome-replay-skill -a opencode

# Global
npx skills add zc277584121/chrome-replay-skill -a opencode -g
```

### Other Agents

`npx skills` supports 40+ agents. Use `-a <agent-name>` to target any supported agent:

```bash
npx skills add zc277584121/chrome-replay-skill -a <agent-name>
```

Common agent names: `windsurf`, `github-copilot`, `cline`, `roo`, `gemini-cli`, `goose`, `kilo`, `augment`.

> **Project vs Global**: Project-level installs the skill into the current project directory (e.g., `.claude/skills/`). Global (`-g`) installs to your home directory (e.g., `~/.claude/skills/`) so it's available across all projects.

## Requirements

Chrome DevTools MCP must be configured and connected. See `context.md` for setup instructions, or refer to the [official Chrome DevTools MCP repository](https://github.com/ChromeDevTools/chrome-devtools-mcp).
