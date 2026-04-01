# GT CLI Guide

`gt` is a command-line tool for managing GuidedTrack programs from the terminal. It handles pushing, pulling, creating, building, and inspecting programs without needing the browser.

## When to Use GT CLI

- **Syncing local `.gt` files with the server** — push local changes up or pull remote source down.
- **Creating new programs** — create programs on the server from local files.
- **Building (compiling) programs** — trigger server-side compilation and see errors.
- **Inspecting programs** — list, search, view source, or download data.
- **Making raw API calls** — use `gt request` for anything the named commands don't cover.

## Installation

```bash
# Install globally via npm
npm install -g @jrc03c/gt-cli

# Verify
gt --version
```

## Authentication

Credentials are resolved in this order:

1. **Environment variables**: `GT_EMAIL` and `GT_PASSWORD`
2. **Config file**: `email` and `password` fields in `gt.config.json`
3. **Interactive prompt**: Falls back to terminal input if neither is set

For non-interactive use (scripts, CI), set environment variables or put credentials in `gt.config.json`.

## Environment

Set `GT_ENV` to target different servers:

| Value | URL |
|-------|-----|
| `production` (default) | `https://www.guidedtrack.com` |
| `stage` | `https://guidedtrack-stage.herokuapp.com` |
| `development` | `https://localhost:3000` |

## Project Setup

### Initialize a project

```bash
gt init
```

Scans the current directory for `.gt` files and interactively links each one to a program on the server (by title, ID, or key). This command requires interactive terminal input, so it cannot be used in non-interactive contexts. As an alternative, you can create `gt.config.json` manually (see format below) using the program ID from `gt program find` or `gt program get`. Creates `gt.config.json`:

```json
{
  "email": "user@example.com",
  "password": "secret",
  "programs": {
    "my-program": {
      "file": "my-program.gt",
      "id": 12345
    }
  }
}
```

The `programs` keys are arbitrary — typically the filename without extension.

### View current config

```bash
gt config
```

## Core Workflow Commands

### Push local files to the server

```bash
# Push all configured programs
gt push

# Push a specific program (by config key)
gt push --only my-program

# Push and build (compile) afterward
gt push --build
```

**Note**: `gt push --build` may fail with a 404 if the config key doesn't match the program's short key. If this happens, push and build separately: `gt push && gt program build <name>`.

### Pull server source to local files

```bash
# Pull all configured programs
gt pull

# Pull a specific program
gt pull --only my-program
```

### Create new programs on the server

```bash
# Create programs by name
gt create my-program another-program

# Create programs for all .gt files in the current directory
gt create
```

### Build (compile) programs

```bash
# Build all programs listed in .gt_projects file
gt build
```

The `.gt_projects` file is a plain text file with one program name per line. The build command polls the server until compilation finishes and reports any errors.

## Program Management

All program commands accept a program name, ID, or key as the identifier.

### List and search

```bash
# List all programs
gt program list

# Search by name
gt program find "my survey"
```

Output format: `{id}\t{key}\t{name}` (tab-separated).

### Inspect

```bash
# Get program metadata as JSON
gt program get my-program

# Get just the source code (useful for piping)
gt program source my-program
```

### Build a specific program

```bash
gt program build my-program
```

### Delete

```bash
# Delete with confirmation prompt
gt program delete my-program

# Skip confirmation
gt program delete my-program --yes
```

### Download data

```bash
# Print CSV to stdout
gt program data my-program

# Save to file
gt program data my-program --output results.csv
```

`gt program csv` is an alias for `gt program data`.

## Browser Shortcuts

These commands open the relevant page in the default browser:

```bash
gt program view my-program      # Opens edit page
gt program preview my-program   # Opens preview (no data saved)
gt program run my-program       # Opens run page (saves data)
```

## Raw API Requests

For endpoints not covered by named commands:

```bash
# GET request
gt request /programs.json

# POST with data
gt request /programs -X POST -d '{"name": "new-program"}'

# Custom headers
gt request /some/endpoint -H "X-Custom: value"
```

Authentication headers are added automatically.

## Key Concepts

- **Program ID**: Numeric identifier from edit URLs (e.g., `9197`).
- **Program key**: 7-character alphanumeric string from run/preview URLs (e.g., `i1qsozk`).
- **Jobs**: Create and build operations are async. The CLI polls `/delayed_jobs/{job_id}` until completion (default: 3-second intervals).
- **`gt.config.json`**: Per-project config linking local files to server programs. Should be gitignored if it contains credentials.
- **`.gt_projects`**: Plain text list of program names used by `gt build`.

## Typical Workflows

### Start a new project from scratch

```bash
# Create .gt files locally, then:
gt create
gt init
gt push --build
```

### Edit locally, sync to server

```bash
# After editing .gt files:
gt push --build
```

### Pull remote changes for local editing

```bash
gt pull
```

### Download experiment data

```bash
gt program data my-survey --output data.csv
```

## Command Reference

| Command | Description |
|---------|-------------|
| `gt init` | Link local `.gt` files to server programs |
| `gt config` | Print current `gt.config.json` |
| `gt push` | Upload local files to server |
| `gt pull` | Download server source to local files |
| `gt create [names...]` | Create new programs on server |
| `gt build` | Compile programs listed in `.gt_projects` |
| `gt program list` | List all programs |
| `gt program find <query>` | Search programs by name |
| `gt program get <name>` | Show program metadata as JSON |
| `gt program source <name>` | Print program source to stdout |
| `gt program build <name>` | Build a specific program |
| `gt program delete <name>` | Delete a program |
| `gt program data <name>` | Download program data as CSV |
| `gt program view <name>` | Open edit page in browser |
| `gt program preview <name>` | Open preview in browser |
| `gt program run <name>` | Open run page in browser |
| `gt request <path>` | Make a raw API request |
| `gt compare [args...]` | Delegate to `gt-compare` tool |
