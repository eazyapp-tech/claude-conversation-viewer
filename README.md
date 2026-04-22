# Claude Code Conversation Viewer

A single-file, zero-dependency GUI for browsing your [Claude Code](https://docs.anthropic.com/en/docs/claude-code) conversation history.

![Python 3.7+](https://img.shields.io/badge/python-3.7%2B-blue)
![No Dependencies](https://img.shields.io/badge/dependencies-none-green)
![Cross Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Windows%20%7C%20Linux-lightgrey)

## Features

- **Conversation browser** -- lists all past Claude Code conversations with search, project filter, and sort
- **Full conversation viewer** -- chat-style display with markdown rendering, syntax-highlighted code blocks, and collapsible tool use/result sections
- **Export** -- download any conversation as `.md` or `.json`
- **Usage stats** -- token counts, model usage breakdown, conversations per project
- **Responsive** -- works on desktop, tablet, and mobile browsers
- **Cross-platform** -- auto-detects `~/.claude/` on macOS/Linux and `%USERPROFILE%\.claude\` on Windows

## Quick Start

```bash
# Clone
git clone https://github.com/eazyapp-tech/claude-conversation-viewer.git
cd claude-conversation-viewer

# Run
python3 claude_conversation_viewer.py
```

Your browser will open automatically at `http://127.0.0.1:5005`.

### Options

```
python3 claude_conversation_viewer.py --port 8080    # custom port
python3 claude_conversation_viewer.py --no-open      # don't auto-open browser
```

## Requirements

- Python 3.7+ (uses only the standard library -- no `pip install` needed)
- A browser
- Existing Claude Code conversations in `~/.claude/projects/`

## How It Works

Claude Code stores conversations as JSONL files in `~/.claude/projects/<project>/<session-id>.jsonl`. This tool:

1. Scans all project directories for conversation files
2. Parses metadata (title, timestamps, models, token usage) from each file
3. Serves a web UI on localhost for browsing and reading conversations
4. All data stays local -- nothing is sent anywhere

## License

MIT
