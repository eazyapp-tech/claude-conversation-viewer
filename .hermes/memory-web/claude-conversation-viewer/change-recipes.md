# Common Change Recipes

Generated: `2026-06-03T17:12:28+05:30`
Memory-web: `claude-conversation-viewer`

> Repo-local AI engineering reference. Verify current source before editing. No secrets are intentionally stored here.

## Before changing code

1. Open [README.md](README.md), [architecture.md](architecture.md), and the relevant module page.
2. Verify source anchors in the current branch.
3. Make changes in an isolated branch/worktree.
4. Run the closest targeted test/build/smoke command.
5. Update this memory-web if the architecture/module map changes.

## Runbook hints from existing docs

<!-- Populated from real source analysis on 2026-06-03T14:59:35+05:30; base origin/main. Do not store secrets here. -->

# Runbooks

## Install from PyPI

```bash
pip install claude-conversation-viewer
```

## Install from Source

```bash
git clone https://github.com/eazyapp-tech/claude-conversation-viewer.git
cd claude-conversation-viewer
pip install .
```

## Run Without Installing

```bash
python3 claude_conversation_viewer.py          # Web UI
python3 claude_conversations_cli.py            # CLI
```

## Start Web UI

```bash
claude-conversations                           # default port 5005
claude-conversations --port 8080               # custom port
claude-conversations --no-open                 # headless / server mode
```

## Start CLI

```bash
claude-conversations-cli                       # interactive
claude-conversations-cli --list                # non-interactive list
claude-conversations-cli --search "auth"       # search
claude-conversations-cli --view <session-id>   # view conversation
claude-conversations-cli --resume <session-id> # resume in Claude Code
```

## Install as macOS Background Service

```bash
claude-conversations --install                 # installs LaunchAgent at ~/Library/LaunchAgents/com.claude-conversation-viewer.plist
claude-conversations --uninstall               # removes LaunchAgent
launchctl list | grep claude-conversation      # check status
```

Service logs to `/tmp/claude-conversation-viewer.log`.

## Update

```bash
pip install --upgrade claude-conversation-viewer
```

## Build Distribution (Maintainers)

```bash
pip install build twine
python3 -m build
# creates dist/claude_conversation_viewer-X.Y.Z.tar.gz and .whl
```

## Publish to PyPI (Maintainers)

```bash
# Test first
python3 -m twine upload --repository testpypi dist/*
# Production
python3 -m twine upload dist/*
```

## Verify Installed Version

```bash
python3 -c "from claude_conversation_viewer import __version__; print(__version__)"
```

## Troubleshoot: No Conversations Found

```bash
ls ~/.claude/projects/
find ~/.claude/projects -name "*.jsonl" | head -5
```

## Troubleshoot: Port Already in Use

```bash
lsof -i :5005
kill <PID>
claude-conversations --port 8080
```

## Troubleshoot: Command Not Found After pip install

```bash
python3 -m site --user-base
export PATH="$(python3 -m site --user-base)/bin:$PATH"
# Or run directly:
python3 -m claude_conversation_viewer.web
python3 -m claude_conversation_viewer.cli
```
