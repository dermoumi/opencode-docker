# Agent Guidelines for vibe

This repository contains a Docker-based development environment for running AI coding assistants ([opencode](https://opencode.ai) and [Claude Code](https://claude.ai)) in containers.

## Project Overview

- **Purpose**: Provides a containerized environment to run opencode and Claude Code with proper user permissions, SSH agent forwarding, GPG forwarding, and VS Code integration
- **Main file**: `vibe` - A bash script that sets up and runs Docker containers
- **Language**: Bash script with Docker Compose inline Dockerfile

---

## Build/Lint/Test Commands

### Running the Script

```bash
# Basic usage - starts a container and runs the last-used tool (default: opencode)
./vibe

# Use opencode explicitly
./vibe -o

# Use Claude Code
./vibe -c

# Build the container before starting
./vibe -b

# Start web UI instead of TUI
./vibe -w

# Start web UI on a custom port
./vibe -w -p 8080

# Open VS Code attached to the container
./vibe -v

# Debug mode - shell into the container instead of running a tool
./vibe -D

# Use local Docker volumes instead of host config directories
./vibe -l

# Tear down (stop and remove containers)
./vibe -d

# Pass extra args to the tool (overrides default --continue)
./vibe -- --help

# Use a different working directory
./vibe /path/to/project
```

### Linting

This project uses ShellCheck for bash linting:

```bash
# Install shellcheck (Arch Linux)
sudo pacman -S shellcheck

# Run shellcheck on the script
shellcheck vibe
```

### Testing

No formal test suite exists. Manual testing involves:
- Running `./vibe` and verifying the container starts
- Verifying the selected tool executes inside the container
- Testing SSH agent forwarding works
- Testing VS Code integration via `-v` flag

### Docker Commands

```bash
# Manual docker compose commands (if needed)
docker compose --project-name vibe-<project>-<hash> up -d
docker compose --project-name vibe-<project>-<hash> down
docker compose --project-name vibe-<project>-<hash> exec vibe bash
```

---

## Code Style Guidelines

### Shell/Bash Conventions

- **Shebang**: Use `#!/bin/bash` (not `#!/bin/sh` for bash-specific features)
- **Error handling**: Always use `set -e` to exit on errors
- **Quotes**: Use double quotes for variable expansions (`"$VAR"`) to prevent word splitting
- **Variables**: Use UPPER_CASE for environment variables, lower_case for local variables
- **Functions**: Define functions before use; use `function_name()` syntax

### Formatting

- Indent with 2 spaces
- Maximum line length: 100 characters (soft limit)
- Use heredocs for multi-line strings (as shown in the docker-compose inline)
- Empty lines between logical sections

### Naming Conventions

- Variables: `VARIABLE_NAME` (uppercase with underscores)
- Constants: Same as variables but readonly where appropriate
- Function names: `function_name` (lowercase with underscores)
- Project names: `vibe-<project>-<hash>`

### Error Handling

```bash
# Exit on any command failure
set -e

# Optional: Exit on undefined variables
set -u

# Optional: Pipe failure detection
set -o pipefail
```

### Docker Best Practices

- Use official base images when possible
- Create non-root users with specific UID/GID
- Mount volumes with proper permissions
- Forward SSH_AUTH_SOCK for git/ssh authentication

### Import Conventions

- No external imports required for this simple script
- All dependencies are handled via Docker

### Single Script Rule

- Always keep all code in a single script file
- Never create external files for code (e.g., separate Dockerfile, docker-compose.yml, helper scripts)
- Use heredocs to embed any additional files inline within the main script
- Avoid suggesting or implementing file-based configuration that could be inlined

---

## Working with This Repository

### Making Changes

1. Test changes manually using `./vibe -D` for debugging
2. Run shellcheck: `shellcheck vibe`
3. Verify docker syntax if modifying the inline Dockerfile

### Adding Features

- Keep the script modular with clear sections
- Document any new flags in the usage message
- Test with different directory names to ensure hashing works correctly

### Common Issues

- **Permission denied**: Ensure the script is executable (`chmod +x vibe`)
- **Docker not running**: Start Docker daemon first
- **Port conflicts**: The project name includes a hash to avoid conflicts; use `-p PORT` to override the default web UI port

---

## Notes for AI Agents

- This is a thin wrapper around Docker Compose - avoid over-engineering
- The inline Dockerfile is embedded as a heredoc for portability
- Always keep all code in a single script file using heredocs
- Never suggest creating separate files (Dockerfile, docker-compose.yml, etc.)
- Changes to the Dockerfile require rebuilding with `-b` flag
- The script is designed to be portable across different host directories
- Tool choice (`-c` for Claude, `-o` for opencode) is persisted at `~/.local/state/vibe/last-tool`
- Inline scripts (entrypoint, code wrapper) are passed as build-args and consumed in the Dockerfile via `ARG` + `RUN echo`
- All `docker exec` invocations must go through `/entrypoint.sh` so environment variables are properly set up
