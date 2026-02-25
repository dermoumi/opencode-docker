# Agent Guidelines for opencode-docker

This repository contains a Docker-based development environment for running [opencode](https://opencode.ai), an AI coding assistant.

## Project Overview

- **Purpose**: Provides a containerized environment to run opencode with proper user permissions and SSH agent forwarding
- **Main file**: `ocode` - A bash script that sets up and runs Docker containers
- **Language**: Bash script with Docker Compose inline Dockerfile

---

## Build/Lint/Test Commands

### Running the Script

```bash
# Basic usage - starts a container and runs opencode
./ocode

# Build the container before starting
./ocode -b

# Debug mode - shell into the container instead of running opencode
./ocode -D

# Tear down (stop and remove containers)
./ocode -d
```

### Linting

This project uses ShellCheck for bash linting:

```bash
# Install shellcheck (Arch Linux)
sudo pacman -S shellcheck

# Run shellcheck on the script
shellcheck ocode
```

### Testing

No formal test suite exists. Manual testing involves:
- Running `./ocode` and verifying the container starts
- Verifying opencode executes inside the container
- Testing SSH agent forwarding works

### Docker Commands

```bash
# Manual docker compose commands (if needed)
docker compose --project-name opencode-<project>-<hash> up -d
docker compose --project-name opencode-<project>-<hash> down
docker compose --project-name opencode-<project>-<hash> exec opencode bash
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
- Project names: `opencode-<project>-<hash>`

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

1. Test changes manually using `./ocode -D` for debugging
2. Run shellcheck: `shellcheck ocode`
3. Verify docker syntax if modifying the inline Dockerfile

### Adding Features

- Keep the script modular with clear sections
- Document any new flags in the usage message
- Test with different directory names to ensure hashing works correctly

### Common Issues

- **Permission denied**: Ensure the script is executable (`chmod +x ocode`)
- **Docker not running**: Start Docker daemon first
- **Port conflicts**: The project name includes a hash to avoid conflicts

---

## Notes for AI Agents

- This is a thin wrapper around Docker Compose - avoid over-engineering
- The inline Dockerfile is embedded as a heredoc for portability
- Always keep all code in a single script file using heredocs
- Never suggest creating separate files (Dockerfile, docker-compose.yml, etc.)
- Changes to the Dockerfile require rebuilding with `-b` flag
- The script is designed to be portable across different host directories
