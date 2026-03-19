# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Conventional Commits](https://www.conventionalcommits.org/).

## [Unreleased]

### Changed

- Extracted inline bash from `build.yml` and `check-upstream.yml` into Taskfile tasks (`build:compile`, `build:package`, `build:release`, `check-upstream`)
- All three workflows now use `jdx/mise-action@v3` + `task` as thin orchestration wrappers

## [0.1.0] - 2026-02-26

### Added

- Build and Release workflow: cross-compiles Rust tools for macOS arm64, Linux x86_64, and Linux arm64 ([#1](https://github.com/prettygood-software/nix-tool-bin-template/pull/1))
- Check Upstream Release workflow: weekly cron to detect new upstream versions ([#1](https://github.com/prettygood-software/nix-tool-bin-template/pull/1))
- Lint workflow with `jdx/mise-action@v3` and `task lint` ([#1](https://github.com/prettygood-software/nix-tool-bin-template/pull/1), [#3](https://github.com/prettygood-software/nix-tool-bin-template/pull/3))
- `tool.json` configuration schema for defining which tool to build ([#1](https://github.com/prettygood-software/nix-tool-bin-template/pull/1))
- mise tool management with 3-tier profiles: base, development, CI ([#2](https://github.com/prettygood-software/nix-tool-bin-template/pull/2))
- Taskfile with `setup` and `lint` tasks ([#2](https://github.com/prettygood-software/nix-tool-bin-template/pull/2))
- lefthook git hooks: file hygiene, YAML linting, commitlint ([#2](https://github.com/prettygood-software/nix-tool-bin-template/pull/2))
- CLAUDE.md and AGENTS.md project documentation

### Fixed

- yamllint now ignores `node_modules/` ([#3](https://github.com/prettygood-software/nix-tool-bin-template/pull/3))
- Removed incorrect required `crate` field from CI lint check ([#3](https://github.com/prettygood-software/nix-tool-bin-template/pull/3))
