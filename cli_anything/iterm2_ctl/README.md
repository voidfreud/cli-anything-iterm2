# cli-anything-iterm2

A stateful CLI harness for iTerm2 — control a running iTerm2 instance programmatically via the command line.

## Prerequisites

### 1. macOS + iTerm2
```bash
# Download from https://iterm2.com/ or via Homebrew:
brew install --cask iterm2
```

### 2. Enable the Python API in iTerm2
Open iTerm2, then:
```
Preferences → General → Magic → Enable Python API ✓
```

### 3. Python requirements
```bash
pip install iterm2 click prompt-toolkit
```

## Installation

```bash
pip install git+https://github.com/voidfreud/cli-anything-iterm2.git
```

Or from source:

```bash
git clone https://github.com/voidfreud/cli-anything-iterm2.git
cd cli-anything-iterm2
pip install -e .
```

This installs the `cli-anything-iterm2` command in your PATH.

## Verify Installation

```bash
which cli-anything-iterm2
cli-anything-iterm2 --help
```

## Usage

**iTerm2 must be running for all commands.**

### Interactive REPL (default)
```bash
cli-anything-iterm2
```

### One-shot commands

```bash
# App status & workspace orientation
cli-anything-iterm2 --json app snapshot               # rich: path, process, role, last output per pane
cli-anything-iterm2 app status
cli-anything-iterm2 app current       # get focused window/tab/session
cli-anything-iterm2 app context       # show saved context

# Window management
cli-anything-iterm2 window list
cli-anything-iterm2 window create --profile "Default"
cli-anything-iterm2 window create --command "python3"
cli-anything-iterm2 window activate <window-id>
cli-anything-iterm2 window set-title "My Window"
cli-anything-iterm2 window fullscreen on
cli-anything-iterm2 window frame

# Tab management
cli-anything-iterm2 tab list
cli-anything-iterm2 tab create --window-id <id>
cli-anything-iterm2 tab activate <tab-id>
cli-anything-iterm2 tab close <tab-id>

# Session (pane) management
cli-anything-iterm2 session list
cli-anything-iterm2 session send "echo hello"              # sends with newline
cli-anything-iterm2 session send "text" --no-newline       # no newline
cli-anything-iterm2 session screen                          # read terminal output
cli-anything-iterm2 session screen --lines 20
cli-anything-iterm2 session split                           # horizontal split
cli-anything-iterm2 session split --vertical               # vertical split
cli-anything-iterm2 session set-name "API Worker"
cli-anything-iterm2 session restart
cli-anything-iterm2 session resize --columns 220 --rows 50
cli-anything-iterm2 session get-var hostname
cli-anything-iterm2 session set-var user.myvar "hello"
cli-anything-iterm2 session selection                       # get selected text

# Profile management
cli-anything-iterm2 profile list
cli-anything-iterm2 profile color-presets
cli-anything-iterm2 profile apply-preset "Solarized Dark"

# Arrangements
cli-anything-iterm2 arrangement list
cli-anything-iterm2 arrangement save "my-layout"
cli-anything-iterm2 arrangement restore "my-layout"
```

### JSON output (for agent use)

Every command supports `--json` for machine-readable output:

```bash
cli-anything-iterm2 --json app status
cli-anything-iterm2 --json window list
cli-anything-iterm2 --json session send "ls -la"
```

## Stateful Context

The CLI saves a context (window_id, tab_id, session_id) to
`~/.cli-anything-iterm2/session.json`. Once set, you can omit IDs:

```bash
# Set context to the currently focused window/tab/session
cli-anything-iterm2 app current

# Now these work without --session-id:
cli-anything-iterm2 session send "echo hello"
cli-anything-iterm2 session screen
cli-anything-iterm2 session split --vertical
```

## Running Tests

```bash
cd cli-anything-iterm2

# Unit tests (no iTerm2 needed)
python3 -m pytest cli_anything/iterm2_ctl/tests/test_core.py -v

# E2E tests (iTerm2 must be running)
python3 -m pytest cli_anything/iterm2_ctl/tests/test_full_e2e.py -v -s

# All tests with installed command verification
CLI_ANYTHING_FORCE_INSTALLED=1 python3 -m pytest cli_anything/iterm2_ctl/tests/ -v -s
```

## Architecture

```
cli-anything-iterm2 (Click CLI)
    ↓
iterm2 Python API (asyncio + WebSocket)
    ↓
iTerm2.app (running macOS terminal emulator)
```

All iTerm2 API calls are async. The harness uses `iterm2.run_until_complete()` to
bridge async operations into Click's synchronous command model.

## Session Variables

iTerm2 sessions expose variables you can read/write:

```bash
cli-anything-iterm2 session get-var hostname      # current hostname
cli-anything-iterm2 session get-var username      # current user
cli-anything-iterm2 session get-var path          # current directory
cli-anything-iterm2 session get-var pid           # shell PID
```

Custom variables use a `user.` prefix:
```bash
cli-anything-iterm2 session set-var user.project "myapp"
cli-anything-iterm2 session get-var user.project
```

**Workspace labeling convention** — set `user.role` on panes when building a workspace so `app snapshot` can identify them:
```bash
cli-anything-iterm2 session set-var user.role "api-server"
cli-anything-iterm2 session set-var user.role "log-tail"
cli-anything-iterm2 session set-var user.role "editor"

# Later — orient in one call
cli-anything-iterm2 --json app snapshot
```
