# nix-tool-bin-template

Template for repos that build and publish prebuilt nix tool binaries.

## Usage

1. Create a new repo from this template
2. Edit `tool.json` with the tool's details:
   - `name`: binary name (what gets installed)
   - `crate`: crates.io package name
   - `upstream`: GitHub URL of the upstream project
   - `description`: one-line description
3. Run the **Build and Release** workflow with the desired version

## How it works

- **Build and Release** (`workflow_dispatch`): Cross-compiles from crates.io for macOS (arm64) and Linux (x86_64, arm64), publishes as GitHub Release with checksums
- **Check Upstream Release** (weekly): Compares upstream releases against ours, creates an issue when a new version is available
- **Lint**: Validates `tool.json` schema and workflow YAML

## mise integration

Once a release is published, the tool can be installed via mise:

```toml
[tools]
"github:prettygood-software/nix-TOOLNAME-bin" = "latest"
```

mise auto-detects GitHub releases with platform-specific tarballs.

## Development

Prerequisites: [mise](https://mise.jdx.dev/)

```bash
# Install tools and git hooks
task setup

# Run all linters (actionlint + yamllint + tool.json validation)
task lint
```
