<!-- Populated from real source analysis on 2026-06-03T14:59:35+05:30; base origin/main. Do not store secrets here. -->

# Source Graph Overview

## Entrypoints (Hotspots)

| File | Role | Importance |
|---|---|---|
| `claude_conversation_viewer/web.py` | Web server + data layer + embedded SPA | Critical — largest file, owns `ConversationStore`, all API routes, JSONL parsing for web |
| `claude_conversation_viewer/cli.py` | Terminal interface + own data layer | Critical — independent JSONL parsing, interactive REPL, ANSI rendering |
| `claude_conversation_viewer/update_checker.py` | PyPI version check | Shared utility; imported by both `web.py` and `cli.py` |
| `claude_conversation_viewer/__init__.py` | Package init + version string | Leaf dependency; imported by `update_checker.py` |

## Dependency Graph

```
claude_conversation_viewer.py   (root wrapper)
    └── claude_conversation_viewer.web:main

claude_conversations_cli.py     (root wrapper)
    └── claude_conversation_viewer.cli:main

web.py
    ├── update_checker.check_for_update_sync()  (optional import)
    └── __init__.__version__  (via update_checker)

cli.py
    ├── update_checker.check_for_update_async()  (optional import)
    └── __init__.__version__  (via update_checker)

update_checker.py
    └── __init__.__version__
```

All imports of `update_checker` are guarded with `try/except ImportError`, making both interfaces functional as standalone scripts.

## Key Data Structures

- **Conversation metadata dict** — produced by `parse_conversation_metadata()`; fields: `id`, `project`, `project_path`, `title`, `has_ai_title`, `first_timestamp`, `last_timestamp`, `models`, `total_messages`, `user_messages`, `assistant_messages`, `cwd`, `version`, `file_path`, `slug`, `is_continuation` (web version adds chain fields)
- **Message entry dict** — produced by `parse_full_conversation()`; fields: `role`, `timestamp`, `uuid`, `model`, `usage`, `content` (list of typed blocks)
- **Content block types:** `{"type": "text"}`, `{"type": "tool_use"}`, `{"type": "tool_result"}`
- **`ConversationStore`** (web only) — holds `conversations[]`, `by_id{}`, `projects[]`, `_file_map{}`, `chains{}`, `chain_of{}`

## Module Coupling

- `web.py` and `cli.py` are intentionally **loosely coupled** — they share no runtime state and duplicate the JSONL parsing logic
- `update_checker.py` is the only true shared module; both consumers use it identically
- The root wrapper scripts are pure delegation — zero logic

## External I/O

| Target | Direction | Module |
|---|---|---|
| `~/.claude/projects/**/*.jsonl` | Read | `web.py`, `cli.py` |
| `https://pypi.org/pypi/claude-conversation-viewer/json` | Read (HTTP GET) | `update_checker.py` |
| `/tmp/claude-viewer-update-check` | Read/Write | `update_checker.py` |
| `~/Library/LaunchAgents/com.claude-conversation-viewer.plist` | Write (macOS) | `web.py` |
| `claude --resume <id>` (subprocess) | Exec | `cli.py` |
| `localhost:5005` (HTTPServer) | Listen | `web.py` |

## Confidence notes

# Confidence Notes

## High Confidence

- **Package structure and entrypoints** — fully confirmed by `pyproject.toml` `[project.scripts]` and root wrapper files
- **Zero-dependency claim** — confirmed by `pyproject.toml` (no `dependencies` key) and stdlib-only imports across all source files
- **Data source path** (`~/.claude/projects/`) — confirmed by `get_projects_dir()` in both `web.py` and `cli.py`
- **JSONL format fields** — confirmed by parser code and DOCS.md examples
- **Version string** — `1.0.0` in `__init__.py`, consistent with `pyproject.toml`
- **Update checker behavior** — fully readable in `update_checker.py`
- **Chain linking algorithm** — fully readable in `ConversationStore._build_chains()` in `web.py`
- **CLI ANSI/Box rendering** — fully readable in `class C` and `class Box` in `cli.py`

## Medium Confidence

- **Embedded SPA content** — `web.py` is confirmed to embed the SPA, but the actual HTML/CSS/JS content was truncated in the provided file excerpt. The API route table is sourced from DOCS.md, which is authoritative documentation.
- **Token counting fields** — several lines in the parsers were `[REDACTED_SECRET_LINE]` in the provided content, covering `total_input_tokens`, `total_output_tokens`, `total_cache_creation`, `total_cache_read`. Field names are inferred from the returned metadata dict keys visible later in the function.
- **`/api/stats` response shape** — endpoint existence confirmed by DOCS.md; exact response schema not visible (inside the embedded SPA handler in truncated `web.py`)

## Needs Source Verification

- **Full SPA JavaScript logic** (search, filter, sort, markdown rendering, syntax highlighting) — embedded in `web.py` but not provided in the file excerpt
- **macOS LaunchAgent plist generation** — referenced in `--install` flag behavior but the plist template content is in the truncated portion of `web.py`
- **`setup.cfg` contents** — file listed in repo tree but content not provided; assumed to mirror `pyproject.toml` metadata for pip <22 compatibility per DOCS.md
- **`/api/conversations` sort/filter parameters** — query string handling is in the truncated HTTP handler section of `web.py`
- **Session chain data exposed to API** — `ConversationStore.chains` and `chain_of` are built but how they are serialized in `/api/conversations` response is in truncated code

| `GET /api/chain/{id}` | Return a conversation chain by id; confirmed in `claude_conversation_viewer/web.py` path handler |
