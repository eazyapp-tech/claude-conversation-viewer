# Pitfalls and Invariants

Generated: `2026-06-03T17:12:28+05:30`
Memory-web: `claude-conversation-viewer`

> Repo-local AI engineering reference. Verify current source before editing. No secrets are intentionally stored here.

## Existing notes

<!-- Populated from real source analysis on 2026-06-03T14:59:35+05:30; base origin/main. Do not store secrets here. -->

# Architectural Decisions

## 1. Zero External Dependencies

**Decision:** Use only Python stdlib.

**Evidence:** `pyproject.toml` has no `dependencies` field. All imports in `web.py`, `cli.py`, `update_checker.py` are stdlib only. README badges: "No Dependencies".

**Rationale (inferred):** Maximizes portability — installable anywhere Python 3.7+ exists without pip resolving conflicts. Reduces supply-chain risk.

---

## 2. Embedded SPA in `web.py`

**Decision:** The entire browser SPA (HTML/CSS/JS) is embedded as a string literal inside `web.py` rather than served from separate static files.

**Evidence:** `web.py` serves `GET /` from an in-memory string; no `static/` or `templates/` directory exists in the repo tree.

**Rationale (inferred):** Keeps the web UI as a single file deployment — consistent with the zero-install, "just run the .py" goal.

---

## 3. Dual Interface (Web + CLI) with Duplicated Parsers

**Decision:** `web.py` and `cli.py` each contain their own `parse_conversation_metadata()` and `parse_full_conversation()` implementations rather than sharing a common library module.

**Evidence:** Both files define these functions independently. The web version tracks additional fields (`slug`, `ai_title`, `is_continuation`, chain metadata); CLI version is leaner.

**Rationale (inferred):** Allows each interface to evolve independently and be run as a standalone file without importing the other. Trade-off: some logic duplication.

---

## 4. In-Memory Data Store (Web UI)

**Decision:** `ConversationStore` loads and indexes all conversations into memory at server startup; no database or persistent cache.

**Evidence:** `ConversationStore.load()` in `web.py` scans all JSONL files on startup. No SQLite, shelve, or other persistence layer present.

**Rationale (inferred):** Simplifies the architecture (no schema migrations, no cache invalidation). Acceptable because conversation files are small and local.

---

## 5. Two-Tier Session Chain Linking

**Decision:** Chain related sessions using (1) shared `slug` field first, then (2) timestamp proximity fallback (≤1 hour gap, same project).

**Evidence:** `ConversationStore._build_chains()` in `web.py` implements this two-pass algorithm.

**Rationale (inferred):** Claude Code added the `slug` field at some point; older sessions lack it. The timestamp fallback provides best-effort linking for legacy data.

---

## 6. Backward-Compatible Root Wrappers

**Decision:** Keep `claude_conversation_viewer.py` and `claude_conversations_cli.py` as thin shims at the repo root.

**Evidence:** Both files contain only a `from ... import main` + `if __name__ == "__main__": main()` pattern.

**Rationale (inferred):** Users who cloned the repo before it was restructured into a package can still run the original filenames without change.

---

## 7. Update Check Caching to Temp File

**Decision:** Cache PyPI update check results to `/tmp/claude-viewer-update-check` for 1 hour, shared between Web UI and CLI processes.

**Evidence:** `CACHE_FILE = Path(tempfile.gettempdir()) / "claude-viewer-update-check"` in `update_checker.py`.

**Rationale (inferred):** Avoids hammering PyPI on every startup, works across both interface modes without a persistent config location.

---

## 8. Version Defined in Three Places

**Decision:** Version string (`1.0.0`) is maintained in `__init__.py`, `pyproject.toml`, and `setup.cfg` separately.

**Evidence:** DOCS.md publishing guide explicitly states to update all three. No single-source-of-truth tooling (e.g., `setuptools-scm`) is used.

**Rationale (inferred):** Chosen for pip <22 compatibility (`setup.cfg` / `setup.py` shims required). Trade-off: manual version bump in three files per release.

## General invariants

- Do not trust this memory-web over current source.
- Keep generated/cache/build artifacts out of implementation commits unless explicitly needed.
- Re-run project-specific tests or smoke checks after edits.
- Preserve existing auth/config/secrets handling; never hardcode credentials.
