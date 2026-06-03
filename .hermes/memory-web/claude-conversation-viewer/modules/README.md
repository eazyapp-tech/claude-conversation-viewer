# Modules and Topic Map

Generated: `2026-06-03T17:12:28+05:30`
Memory-web: `claude-conversation-viewer`

> Repo-local AI engineering reference. Verify current source before editing. No secrets are intentionally stored here.

## Semantic module map

<!-- Populated from real source analysis on 2026-06-03T14:59:35+05:30; base origin/main. Do not store secrets here. -->

# Modules

## `claude_conversation_viewer/` — Main Package

### `claude_conversation_viewer/__init__.py`
- Exports `__version__ = "1.0.0"`
- Used by `update_checker.py` to compare against PyPI

### `claude_conversation_viewer/web.py`
- **Responsibility:** Web UI server + full data layer for the browser interface
- Key functions:
  - `get_claude_dir()` / `get_projects_dir()` — platform-aware path detection
  - `parse_conversation_metadata(filepath)` — fast JSONL scan for list view; returns dict with id, project, title, timestamps, models, token counts, message counts, chain metadata
  - `parse_full_conversation(filepath)` — full JSONL parse; returns list of message dicts with typed content blocks (`text`, `tool_use`, `tool_result`)
  - `export_as_markdown(filepath, metadata)` — renders conversation to Markdown string
  - `class ConversationStore` — loads all conversations at startup, builds project index, performs chain linking via `_build_chains()`
  - `main()` — argument parsing, server startup, optional browser open, optional macOS LaunchAgent install/uninstall
- The SPA (HTML/CSS/JS) is embedded directly in this file (not shown in truncated content)

### `claude_conversation_viewer/cli.py`
- **Responsibility:** Terminal CLI — interactive REPL and non-interactive flag modes
- Key components:
  - `class C` — ANSI color codes, TTY-aware
  - `class Box` — Unicode box-drawing characters
  - `get_claude_dir()` / `get_projects_dir()` — duplicated path detection (independent of `web.py`)
  - `parse_conversation_metadata(filepath)` — lighter version of the web parser (no chain/slug/ai-title fields)
  - `parse_full_conversation(filepath)` — similar to web version, 2000-char truncation on tool results (vs 5000 in web)
  - `load_all_conversations()` — scans and sorts all conversations
  - `print_welcome()` — styled banner with command reference
  - `print_conversation_list()` — paginated, column-aligned table
  - `format_date()` — human-relative timestamps
  - `main()` — argument parsing, interactive loop, non-interactive dispatch

### `claude_conversation_viewer/update_checker.py`
- **Responsibility:** PyPI version check with file-based caching
- Key functions:
  - `_parse_version(v)` — converts semver string to comparable tuple
  - `_read_cache()` / `_write_cache()` — JSON cache at `/tmp/claude-viewer-update-check`, 1-hour TTL
  - `_do_check()` — cache-first PyPI fetch
  - `check_for_update_sync()` — blocking call (used by web.py)
  - `check_for_update_async(callback)` — daemon thread (used by cli.py)

## Root-level Wrappers

### `claude_conversation_viewer.py`
- Backward-compatible entry point → delegates to `claude_conversation_viewer.web:main`
- Allows `python3 claude_conversation_viewer.py` without installation

### `claude_conversations_cli.py`
- Backward-compatible entry point → delegates to `claude_conversation_viewer.cli:main`
- Allows `python3 claude_conversations_cli.py` without installation

## Build / Packaging Files

| File | Purpose |
|---|---|
| `pyproject.toml` | PEP 621 package metadata, entry points, Python >=3.7 constraint |
| `setup.cfg` | setuptools config for pip <22 compatibility |
| `setup.py` | Minimal `setup()` shim for pip <22 compatibility |

## Indexed source groups

| Area | File count | First anchors |
|---|---:|---|
| `DOCS.md` | 1 indexed files | `DOCS.md` |
| `README.md` | 1 indexed files | `README.md` |
| `claude_conversation_viewer.py` | 1 indexed files | `claude_conversation_viewer.py` |
| `claude_conversation_viewer/__init__.py` | 1 indexed files | `claude_conversation_viewer/__init__.py` |
| `claude_conversation_viewer/cli.py` | 1 indexed files | `claude_conversation_viewer/cli.py` |
| `claude_conversation_viewer/update_checker.py` | 1 indexed files | `claude_conversation_viewer/update_checker.py` |
| `claude_conversation_viewer/web.py` | 1 indexed files | `claude_conversation_viewer/web.py` |
| `claude_conversations_cli.py` | 1 indexed files | `claude_conversations_cli.py` |
| `pyproject.toml` | 1 indexed files | `pyproject.toml` |
| `setup.py` | 1 indexed files | `setup.py` |

## Topic pages

- [source-files.md](source-files.md) — full high-signal source file list
- [../core/entrypoints-runtime.md](../core/entrypoints-runtime.md)
- [../core/dependencies-config.md](../core/dependencies-config.md)
