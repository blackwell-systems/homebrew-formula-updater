# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/).

## [v0.1.1] - 2026-03-02

### Fixed

- **Commit message format no longer inferred from git log** — the previous approach used `git log --oneline -5` to infer the tap's commit convention, which is unreliable if recent commits were made by automated tooling with non-standard prefixes. The format is now hardcoded to the standard Homebrew tap convention: `<name> <version-without-v>` (e.g. `claudewatch 1.2.0`), with no `chore:`, `feat:`, or other prefix.

## [v0.1.0] - 2026-03-01

### Added

- Initial release of the homebrew-formula-updater skill
- 8-step pipeline: locate tap, verify tap state, find formula, get checksums, update formula, verify diff, commit and push, report
- Tap discovery checks `TAP_REPO` environment variable, `~/code/<tap-name>`, and `/opt/homebrew/Library/Taps/<org>/<tap-name>` before cloning
- Tap state verification: working tree must be clean, branch must not be behind remote, no rebase or merge in progress
- Checksum sourcing with explicit priority: checksum file attached to release first, checksums passed from `github-release-engineer` second, ask user as last resort
- Formula update limited to three field types only: `version`, `url`, and `sha256`; all other formula structure preserved exactly including whitespace, comments, and platform blocks
- Only updates platforms already present in the formula — does not add new platform blocks
- Commit message format inferred from tap's existing git log (`git log --oneline -5`) to match tap conventions
- Diff verification step before committing: shows `git diff` and requires user confirmation; if anything looks wrong, asks rather than auto-correcting
- Structured report on completion: tap path, version transition, commit SHA, push status, and per-platform SHA256 values
- Error handling policy table covering all failure modes: stop-and-report for structural failures, ask-user for missing checksum data
