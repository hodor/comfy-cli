# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

comfy-cli is a command-line tool for installing, managing, and launching ComfyUI. It provides cross-platform support (Windows, macOS, Linux) for ComfyUI installation, custom node management, model downloading, and PR testing.

**Repository**: https://github.com/Comfy-Org/comfy-cli
**Python**: 3.10+
**CLI Framework**: Typer with Rich for terminal output

## Development Commands

```bash
# Install in development mode
pip install -e .

# Install dev dependencies
pip install -e ".[dev]"

# Set dev environment (enables localhost registry API)
export ENVIRONMENT=dev

# Run tests with coverage
pytest --cov=comfy_cli --cov-report=xml .

# Run specific test file
pytest tests/comfy_cli/test_config_manager.py

# Lint check
ruff check

# Format check
ruff format --diff

# Format code
ruff format

# Install pre-commit hooks
pre-commit install
```

## Architecture

### Entry Points
- `comfy_cli/__main__.py` - Main entry point
- `comfy_cli/cmdline.py` - All CLI commands registered here using Typer

### Core Components

**Workspace Management** (`workspace_manager.py`):
- Detects ComfyUI installations by checking git remote against `COMFY_ORIGIN_URL_CHOICES`
- Manages workspace selection via `--workspace`, `--recent`, `--here` flags
- Reads/writes `comfy.yaml` in workspace root

**Configuration** (`config_manager.py`):
- Singleton managing INI config at platform-specific paths
- Windows: `%LOCALAPPDATA%\comfy-cli\config.ini`
- macOS: `~/Library/Application Support/comfy-cli/config.ini`
- Linux: `~/.config/comfy-cli/config.ini`

**Commands** (`command/` directory):
- `install.py` - ComfyUI installation, GPU setup, version/PR handling
- `launch.py` - Run ComfyUI with optional frontend PR testing
- `custom_nodes/` - Node install/update/snapshot/bisect commands
- `models/` - Model download/list/remove with CivitAI/HuggingFace support
- `pr_command.py` - PR cache management

**Dependency Compilation** (`uv.py`):
- Uses UV for fast dependency resolution
- Compiles requirements for custom nodes
- Handles torch/CUDA version conflicts

**Registry API** (`registry/`):
- Client for Comfy node registry (api.comfy.org)
- Environment-based: dev→localhost, staging→stagingapi.comfy.org, prod→api.comfy.org

### Adding New Commands

Simple command in `cmdline.py`:
```python
@app.command()
def my_command(arg: str):
    """Help text"""
    pass
```

Subcommand group - create `comfy_cli/command/[name]/`:
```python
# __init__.py
from .command import app

# command.py
import typer
app = typer.Typer()

@app.command()
def subcommand():
    pass
```

Then register in `cmdline.py`:
```python
from comfy_cli.command import new_command
app.add_typer(new_command.app, name="new-command")
```

## Key Conventions

- Use `typer` for all CLI argument handling
- Use `rich` for console output and progress bars
- Use `questionary` for interactive prompts
- The `tracking.py` module handles opt-in Mixpanel analytics
- Constants in `constants.py` define GPU options, CUDA versions, default paths

## Debugging

VSCode launch.json:
```json
{
  "name": "Python Debugger: Run",
  "type": "debugpy",
  "request": "launch",
  "module": "comfy_cli.__main__",
  "args": [],
  "console": "integratedTerminal"
}
```

## Testing ComfyUI-Manager Changes

```bash
# Install from fork
comfy --here install --manager-url=<url-or-path>

# Switch manager branch
cd ComfyUI/custom_nodes/ComfyUI-Manager && git checkout <branch>
```

## Custom Node Packaging

`.comfyignore` file (gitignore syntax) excludes files from `comfy node pack`:
```gitignore
docs/
tests/
*.psd
```
