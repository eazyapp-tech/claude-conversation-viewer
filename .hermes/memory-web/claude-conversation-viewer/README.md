# Claude Conversation Viewer Memory Web

Generated: `2026-06-03T17:12:28+05:30`
Memory-web: `claude-conversation-viewer`

> Repo-local AI engineering reference. Verify current source before editing. No secrets are intentionally stored here.

## Start by task type

| Task | Open |
|---|---|
| Architecture / stack | [architecture.md](architecture.md) |
| Main modules and source anchors | [modules/README.md](modules/README.md) |
| Entrypoints and runtime flow | [core/entrypoints-runtime.md](core/entrypoints-runtime.md) |
| Dependencies / config | [core/dependencies-config.md](core/dependencies-config.md) |
| Safe edits | [change-recipes.md](change-recipes.md) |
| Pitfalls | [pitfalls.md](pitfalls.md) |
| Source coverage | [source-coverage-map.md](source-coverage-map.md) |

## Most important facts

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
- Browser is auto-opened via `webbrowser.open()` in a thread unless 

## Source verification habit

This memory-web is a map. Before modifying behavior, open the source anchors listed in the relevant topic page and verify the current implementation.
