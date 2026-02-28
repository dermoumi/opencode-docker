# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A single bash script (`vibe`) that launches Docker containers running [opencode](https://opencode.ai) or [Claude Code](https://claude.ai) as AI coding assistants, with SSH agent forwarding, GPG forwarding, and VS Code integration.

## Commands

```bash
# Lint
shellcheck vibe

# Debug (shell into container instead of running a tool)
./vibe -D

# Rebuild container image
./vibe -b

# Tear down container
./vibe -d
```

## Critical Constraints

**Keep all code in a single file.** Never create separate `Dockerfile`, `docker-compose.yml`, or helper scripts. Use heredocs to embed everything inline. This is the most important rule in this codebase.

**All `docker exec` calls must invoke `/entrypoint.sh` first** so that environment variables (SSH_AUTH_SOCK, GPG socket, EDITOR, gitconfig) are properly configured inside the container.

## Architecture

The script embeds a complete Docker Compose definition (with an inline Dockerfile) as a heredoc in `$DOCKER_COMPOSE`. Key design decisions:

- **Entrypoint and `code` wrapper** are passed as Docker build args (`ENTRYPOINT`, `CODE_WRAPPER`) and written to the image via `RUN echo "$$VAR" > file`. The double-`$$` escapes heredoc interpolation.
- **Project naming**: `vibe-<dirname>-<4char-md5hash>` (lowercased), ensuring uniqueness across different directories.
- **Config persistence**: By default, host config dirs (`~/.claude`, `~/.config/opencode`, etc.) are bind-mounted into the container. With `-l`, named Docker volumes are used instead.
- **VS Code FIFO bridge**: When `code` is available on the host, two FIFOs (`/tmp/vibe-vscode-<project>.fifo` and `…-resp.fifo`) are created and mounted into the container. A background watcher loop reads paths from the FIFO and opens them in VS Code via `vscode-remote://attached-container+<hex-id>`. The `code` wrapper inside the container writes to the FIFO; `--wait`/`-w` causes it to block until the host signals `done`.
- **Tool persistence**: Last-used tool (`opencode` or `claude`) is stored at `~/.local/state/vibe/last-tool`.
- **Extra args**: Arguments after `--` are passed directly to the tool, overriding the default `--continue` flag.

## Docker Compose Variables

Environment variables used in the compose template (set by the script before passing the heredoc via stdin):

| Variable | Purpose |
|---|---|
| `TARGET_DIR` | Host directory bind-mounted to `/workspace` |
| `WEB_PORT` | Port for web UI (default 4096) |
| `GNUPG_SOCK` | Host GPG extra socket path |
| `GITCONFIG` | Host `~/.gitconfig` path |
| `VSCODE_FIFO` / `VSCODE_FIFO_RESP` | FIFO paths for VS Code bridge |
| `OPENCODE_SHARE/STATE/CONFIG` | opencode config dirs (host or volume) |
| `CLAUDE_HOME` | Claude config dir (host or volume) |
