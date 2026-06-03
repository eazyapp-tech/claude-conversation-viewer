# Architecture

Generated: `2026-06-03T17:12:28+05:30`
Memory-web: `claude-conversation-viewer`

> Repo-local AI engineering reference. Verify current source before editing. No secrets are intentionally stored here.

## Architecture summary from existing semantic docs

<!-- Populated from real source analysis on 2026-06-03T14:59:35+05:30; base origin/main. Do not store secrets here. -->

# Architecture: claude-conversation-viewer

## Overview

A zero-dependency, pure Python (3.7+) tool for browsing, searching, and resuming Claude Code conversation history. Ships two independent user-facing interfaces backed by a shared data layer.

## Runtime

- **Language:** Python 3.7+ (stdlib only — no third-party packages)
- **Distribution:** PyPI package (`claude-conversation-viewer`) via setuptools/PEP 621
- **Platforms:** macOS, Linux, Windows (path detection is platform-aware)

## Data Source

Claude Code writes conversation JSONL files to:
- macOS/Linux: `~/.claude/projects/<project-slug>/<session-id>.jsonl`
- Windows: `%USERPROFILE%\.claude\projects\<project-slug>\<session-id>.jsonl`

Each line is a JSON object with `type`, `message` (containing `role`, `content`, `model`, `usage`), `timestamp`, `cwd`, `uuid`, and optionally `slug`, `isMeta`.

## Data Flow

```
~/.claude/projects/**/*.jsonl
        |
        v
parse_conversation_metadata()   <- fast scan (list view)
parse_full_conversation()        <- full parse (detail view)
        |
        v
  ConversationStore (web.py)     <- in-memory store, built at startup
  load_all_conversations() (cli.py)
        |
        +---> Web UI  (HTTPServer on localhost:5005, embedded SPA)
        +---> CLI     (interactive terminal, ANSI-colored)
```

## Entrypoints

| Installed command | Python entrypoint |
|---|---|
| `claude-conversations` | `claude_conversation_viewer.web:main` |
| `claude-conversations-cli` | `claude_conversation_viewer.cli:main` |
| `python3 claude_conversation_viewer.py` | backward-compat wrapper → `web:main` |
| `python3 claude_conversations_cli.py` | backward-compat wrapper → `cli:main` |

## Web UI Architecture

- `web.py` runs a `http.server.HTTPServer` (stdlib) on `localhost:5005` (configurable)
- The entire SPA (HTML/CSS/JS) is embedded as a string inside `web.py` — no static file serving
- API routes handled in `BaseHTTPRequestHandler.do_GET`:
  - `GET /` — serve the SPA
  - `GET /api/conversations` — all metadata + project list
  - `GET /api/conversation/<id>` — full messages for one conversation
  - `GET /api/export/<id>?format=md|json` — download conversation
  - `GET /api/stats` — aggregate usage statistics
  - `GET /api/update-check` — `{update_available, current_version, latest_version}`
- Browser is auto-opened via `webbrowser.open()` in a thread unless `--no-open`
- macOS LaunchAgent support via `--install` / `--uninstall` flags

## CLI Architecture

- `cli.py` runs as an interactive REPL using `input()` / `print()`
- ANSI colors via `class C` (auto-disabled when stdout is not a TTY)
- Box-drawing characters via `class Box`
- Pagination: 20 conversations per page
- Conversation resume: spawns `claude --resume <session-id>` via `subprocess` / `os.execvp`
- Non-interactive mode: `--list`, `--search`, `--view`, `--resume` flags

## Session Chain Linking (`ConversationStore._build_chains`)

Two-tier algorithm to group related sessions:
1. **Slug-based:** conversations sharing the same `slug` field are grouped; root is the non-continuation session
2. **Timestamp proximity:** unlinked continuations are matched to the closest preceding session in the same project within a 1-hour window

## Update Checker

- `update_checker.py` queries `https://pypi.org/pypi/claude-conversation-viewer/json`
- Results cached to `/tmp/claude-viewer-update-check` for 1 hour
- Web UI: invoked synchronously via `check_for_update_sync()`
- CLI: invoked asynchronously via `check_for_update_async(callback)` in a daemon thread
- Fails silently on any network/parse error

## Dependencies

None beyond Python stdlib: `argparse`, `json`, `os`, `platform`, `re`, `sys`, `threading`, `webbrowser`, `datetime`, `http.server`, `pathlib`, `urllib`, `shutil`, `subprocess`, `tempfile`, `time`.

## Runtime/deployment notes

No CLAUDE.md present. Use package/config files and source anchors below.
