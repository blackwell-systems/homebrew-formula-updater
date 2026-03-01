# homebrew-formula-updater

[![Blackwell Systems™](https://raw.githubusercontent.com/blackwell-systems/blackwell-docs-theme/main/badge-trademark.svg)](https://github.com/blackwell-systems)
[![Version](https://img.shields.io/github/v/release/blackwell-systems/homebrew-formula-updater)](https://github.com/blackwell-systems/homebrew-formula-updater/releases/latest)

A Claude Code skill for automating Homebrew tap formula updates as part of a release process. Given a version and per-platform SHA256 checksums, it locates the formula in the tap repository, updates the version string and all hashes, commits the change with a conventional message, and pushes.

## What It Does

Release automation breaks into two problems: publishing artifacts and updating downstream manifests. This skill handles the second half. After binaries are published to GitHub Releases, the Homebrew formula in your tap must be updated to point at the new version and carry correct checksums for each platform artifact. Doing this manually is error-prone and blocks users from installing via `brew upgrade`.

`homebrew-formula-updater` removes the manual step. Given:

- **Version**: the release tag (e.g. `v1.4.2`)
- **Checksums**: a map of platform artifact names to their SHA256 hashes

The skill:

1. Locates the formula file in the tap repository
2. Updates the `version` field
3. Replaces each platform's `sha256` value with the provided checksum
4. Commits the change with message `chore: update formula to <version>`
5. Pushes the commit to the tap's default branch

## How It Composes

`homebrew-formula-updater` is designed to be called by a higher-level release orchestrator — specifically `github-release-engineer` — after platform artifacts have been published and their checksums are known. The orchestrator collects the version, artifact names, and SHA256 values from the release job and passes them to this skill as structured input.

It is also usable standalone. If you have a version string and a set of checksums in hand, you can invoke the skill directly without going through a release pipeline.

The skill makes no assumptions about how the tap is structured beyond the standard `Formula/<name>.rb` convention. It does not build, sign, or upload artifacts. Its scope is strictly formula mutation and push.

## Install as a Claude Code Skill

Copy the skill file to your global Claude Code commands directory:

```bash
cp commands/homebrew-formula-updater.md ~/.claude/commands/homebrew-formula-updater.md
```

Then invoke it from Claude Code:

```
/homebrew-formula-updater --version v1.4.2 --tap owner/homebrew-tap --formula myproject
```

Provide checksums either inline as arguments or by pointing the skill at a checksums manifest file produced by your release job.

### Inputs

| Parameter    | Description                                              |
|--------------|----------------------------------------------------------|
| `--version`  | Release version tag (e.g. `v1.4.2`)                     |
| `--tap`      | GitHub repository for the Homebrew tap (e.g. `org/homebrew-tools`) |
| `--formula`  | Formula name without the `.rb` extension                 |
| `--checksums`| Path to a checksums file, or inline `name=sha256` pairs  |

### When to Use It

Call this skill after `gh release create` (or equivalent) has completed and artifact URLs are live. The checksums must be final — they are written directly into the formula without verification against the remote artifact.

Do not call this skill before artifacts are published. Homebrew will reject a formula whose URLs resolve to 404 or whose checksums do not match the downloaded content.

## Files

- [`prompts/homebrew-formula-updater.md`](prompts/homebrew-formula-updater.md): The `/homebrew-formula-updater` Claude Code skill (copy to `~/.claude/commands/homebrew-formula-updater.md`)

## License

[MIT](LICENSE)
