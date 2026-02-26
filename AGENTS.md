---
last_validated: 2026-02-26T22:09:43Z+00:00
---

# Agent Instructions: nix-tool-bin-template

This file provides guidance to Claude Code when working with code in this repository.

## Repository Overview

Template repository for building and publishing prebuilt Nix tool binaries as GitHub Releases. Each repo created from this template cross-compiles a single Rust-based Nix tool for macOS (arm64) and Linux (x86_64, arm64), making it installable via mise's `github:` backend.

## Repository Structure

```
/Users/michalolechowski/Projects/nix-tool-bin-template
├── .github
│   └── workflows
│       ├── build.yml
│       ├── check-upstream.yml
│       └── lint.yml
├── AGENTS.md
├── CHANGELOG.md
├── CLAUDE.md
├── lefthook
│   ├── commit-msg.yml
│   ├── files.yml
│   └── lint.yml
├── lefthook.yml
├── package.json
├── README.md
├── Taskfile.yml
├── taskfiles
│   ├── build.yml
│   ├── check-upstream.yml
│   ├── lint.yml
│   └── setup.yml
└── tool.json
```

## Key Files

### tool.json

Configuration file that defines which tool to build. Each repo created from this template customizes this:

```json
{
  "name": "tool-name",
  "crate": "crate-name",
  "upstream": "https://github.com/owner/tool",
  "description": "One-line description"
}
```

- `name`: Binary name (what gets installed)
- `crate`: crates.io package name (omit if building from git)
- `source`: Set to `"git"` when the tool isn't on crates.io (e.g., statix)
- `upstream`: GitHub URL of the upstream project
- `description`: One-line description

### GitHub Workflows

All workflows use `jdx/mise-action@v3` for tool installation and delegate logic to Taskfile tasks. Workflows are thin orchestration wrappers — they don't contain business logic.

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `build.yml` | `workflow_dispatch` with version input | Orchestrates `task build:*` — cross-compile, package, release |
| `check-upstream.yml` | Weekly cron + `workflow_dispatch` | Runs `task check-upstream` — compare upstream vs ours, create issue |
| `lint.yml` | PR + push to main | Runs `task lint` — actionlint, yamllint, tool.json validation |

### Build Targets

| Target | Runner | Architecture |
|--------|--------|-------------|
| `aarch64-apple-darwin` | `macos-14` | macOS arm64 |
| `x86_64-unknown-linux-gnu` | `ubuntu-latest` | Linux x86_64 |
| `aarch64-unknown-linux-gnu` | `ubuntu-latest` (cross) | Linux arm64 |

## Dev Tooling

### mise (Tool Version Management)

Three-tier profile system:

| File | Purpose | Tools |
|------|---------|-------|
| `.mise.toml` | Base (always loaded) | task, actionlint, yamllint, jq |
| `.mise.development.toml` | Dev-only (local) | lefthook, node |
| `.mise.ci.toml` | CI profile (empty) | Inherits base only |
| `.miserc.toml` | Sets default env | `env = ["development"]` |

CI workflows use `MISE_ENV=ci` to avoid installing dev-only tools.

### Taskfile

All workflow logic lives in Taskfile tasks — GitHub Actions workflows are thin orchestration wrappers that call `task <name>`.

```bash
task setup              # Install tools + hooks + npm deps
task lint               # Run actionlint + yamllint + tool.json validation
task build:compile      # Compile tool from crates.io (needs CRATE, VERSION, TARGET)
task build:package      # Package binary as tarball + sha256 (needs TOOL_NAME, TARGET)
task build:release      # Create GitHub Release (needs GH_TOKEN, TOOL_NAME, VERSION)
task check-upstream     # Check for new upstream releases (needs GH_TOKEN, GITHUB_REPOSITORY)
```

### lefthook (Git Hooks)

Modular config via `lefthook/*.yml`:

| Hook | Checks |
|------|--------|
| `pre-commit` | Trailing whitespace, end-of-file newline, large files, merge conflicts, actionlint, yamllint, tool.json |
| `commit-msg` | Conventional commits via commitlint |

## Creating a New Tool Repo

1. Create repo from this template (via OpenTofu with `template` block)
2. Customize `tool.json` with the tool's details
3. If tool isn't on crates.io, set `"source": "git"` and modify `build.yml` to use `cargo install --git`
4. Trigger Build and Release workflow with the desired version

## Development Guidelines

### Setup

```bash
task setup  # installs mise tools, lefthook hooks, npm deps
```

### Linting

```bash
task lint   # actionlint + yamllint + tool.json validation
```

### Adding a New Build Target

1. Add entry to the `matrix` in `.github/workflows/build.yml`
2. Ensure the runner supports the target architecture
3. Update release asset naming pattern if needed

### Modifying tool.json Schema

If adding new fields to `tool.json`, update validation in:
- `taskfiles/lint.yml` (local)
- Both use the same `jq` field check loop

## Git Workflow

```bash
git checkout -b feat/description
# Make changes
git add <files>
git commit -m "feat: description"  # commitlint enforces conventional commits
git push -u origin feat/description
gh pr create
```

## Conventions

- Conventional commits enforced by commitlint
- YAML files linted with yamllint (200 char line limit, no document-start required)
- GitHub Actions workflows linted with actionlint
- `node_modules/` excluded from yamllint via `.yamllint.yml`
- Workflow logic in Taskfile tasks, not inline bash — workflows orchestrate only
