# Claude Code Conversation Viewer

Browse, search, and resume your [Claude Code](https://docs.anthropic.com/en/docs/claude-code) conversations -- from the browser or the terminal.

![Python 3.7+](https://img.shields.io/badge/python-3.7%2B-blue)
![No Dependencies](https://img.shields.io/badge/dependencies-none-green)
![Cross Platform](https://img.shields.io/badge/platform-macOS%20%7C%20Windows%20%7C%20Linux-lightgrey)

## Features

- **Conversation browser** -- lists all past conversations with search, project filter, and sort
- **Full conversation viewer** -- chat-style display with markdown rendering and syntax highlighting
- **Resume conversations** -- jump back into any conversation with `claude --resume`
- **Export** -- download any conversation as `.md` or `.json`
- **Usage stats** -- token counts, model usage breakdown, conversations per project
- **Two interfaces** -- Web GUI and Terminal CLI, both zero-dependency
- **Cross-platform** -- auto-detects `~/.claude/` on macOS/Linux and `%USERPROFILE%\.claude\` on Windows

## Quick Start

```bash
git clone https://github.com/eazyapp-tech/claude-conversation-viewer.git
cd claude-conversation-viewer
```

### Web UI

```bash
python3 claude_conversation_viewer.py
```

Opens a browser-based GUI at `http://127.0.0.1:5005` with search, filters, stats dashboard, and export.

```
python3 claude_conversation_viewer.py --port 8080    # custom port
python3 claude_conversation_viewer.py --no-open      # don't auto-open browser
```

### CLI

```bash
python3 claude_conversations_cli.py                          # interactive browser
python3 claude_conversations_cli.py --list                   # print all conversations
python3 claude_conversations_cli.py --search "auth"          # search by keyword
python3 claude_conversations_cli.py --project "myproject"    # filter by project
python3 claude_conversations_cli.py --view <session-id>      # view full conversation
python3 claude_conversations_cli.py --resume <session-id>    # resume in Claude Code
```

In interactive mode:

| Command | Action |
|---------|--------|
| `#number` | View conversation details + session ID |
| `v #number` | View full messages |
| `r #number` | Resume conversation in Claude Code |
| `s <query>` | Search conversations |
| `n` / `p` | Next / previous page |
| `q` | Quit |

Partial session IDs work too -- `--view 4925f6c7` matches `4925f6c7-35f2-4340-bad6-ad59d4d724ee`.

## Requirements

- Python 3.7+ (standard library only -- no `pip install` needed)
- Existing Claude Code conversations in `~/.claude/projects/`
- Claude Code CLI installed (for `--resume`)

## How It Works

Claude Code stores conversations as JSONL files in `~/.claude/projects/<project>/<session-id>.jsonl`. These tools:

1. Scan all project directories for conversation files
2. Parse metadata (title, timestamps, models, token usage) from each file
3. Present them in a browsable interface (web or terminal)
4. All data stays local -- nothing is sent anywhere

## License

MIT
