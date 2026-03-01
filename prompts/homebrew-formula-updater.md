<!-- homebrew-formula-updater v0.1.0 -->
Homebrew Formula Updater

You are updating a Homebrew tap formula to point to a new release. Work through the steps below in order. Each step gates the next.

## Inputs

Accept the following inline or ask interactively — all at once, not one at a time:

| Input | Description |
|---|---|
| `version` | The release version, e.g. `v1.2.0` |
| `repo` | The GitHub repo being released, e.g. `blackwell-systems/claudewatch` |
| `tap` | The tap repo, e.g. `blackwell-systems/homebrew-tap` (default: infer from org) |

When called from `github-release-engineer`, these values come from the step report. When called standalone, ask the user.

Confirm inputs before proceeding:

```
Updating: <tap>/Formula/<name>.rb
Version:  <version>
Repo:     https://github.com/<repo>
```

Wait for confirmation.

## Step 1 — Locate the tap

Find the tap repository on disk. Check in order:

1. A path configured via `TAP_REPO` environment variable
2. `~/code/<tap-name>` (e.g. `~/code/homebrew-tap`)
3. `/opt/homebrew/Library/Taps/<org>/<tap-name>`

If not found locally, clone it:

```bash
gh repo clone <tap> /tmp/<tap-name>
```

Store the tap path. All subsequent file operations use this path.

## Step 2 — Verify tap state

Before touching anything:

```bash
cd <tap-path>
git fetch origin
git status
```

- Working tree must be clean. If not, stop and report.
- Local branch must not be behind remote. If behind, `git pull --ff-only`. If that fails, stop and report.
- No rebase or merge in progress. Check for `.git/MERGE_HEAD`, `.git/rebase-merge/`, `.git/rebase-apply/`. Stop and report if found.

## Step 3 — Find the formula

Look for the formula file at `<tap-path>/Formula/<name>.rb` where `<name>` is the repo name (last path component of `repo`, lowercased).

If not found, check `<tap-path>/Casks/` as a fallback. If still not found, stop and report — do not create a new formula file.

## Step 4 — Get checksums

Checksums must come from the GitHub release. Do not download artifacts locally.

**Primary: checksum file attached to release**

Look for an asset on the release whose name matches `*checksums*`, `*sha256*`, or `*SHA256SUMS*`:

```bash
gh release view <version> --repo <repo> --json assets --jq '.assets[] | select(.name | test("checksums|sha256|SHA256SUMS"; "i")) | .url'
```

If found, download and parse it. goreleaser-format checksum files look like:

```
abc123...  claudewatch_1.2.0_darwin_arm64.tar.gz
def456...  claudewatch_1.2.0_darwin_amd64.tar.gz
...
```

Extract the SHA256 for each platform artifact.

**Fallback: checksums passed from release engineer**

If no checksum file is attached, look for checksums in the inputs passed from `github-release-engineer`. If present, use them directly.

**Last resort: ask the user**

If neither source provides checksums, report what was found and ask the user to provide the SHA256 values for each platform.

Required platforms (check which are present in the current formula and match accordingly):

- `darwin_arm64`
- `darwin_amd64`
- `linux_arm64`
- `linux_amd64`

Only update platforms that already exist in the formula. Do not add new platform blocks.

## Step 5 — Update the formula

Read the formula file. Make three types of changes:

**Version string:**

Find the `version "..."` line and replace the version value. Strip the `v` prefix if the current formula does not use it; keep it if it does. Match the existing format exactly.

**URLs:**

For each platform block, find the `url "..."` line and replace it with the new release URL. Construct URLs from the pattern already present in the formula — replace only the version component. Example:

```
url "https://github.com/blackwell-systems/claudewatch/releases/download/v0.4.2/claudewatch_0.4.2_darwin_arm64.tar.gz"
```

becomes:

```
url "https://github.com/blackwell-systems/claudewatch/releases/download/v1.2.0/claudewatch_1.2.0_darwin_arm64.tar.gz"
```

**SHA256 hashes:**

For each platform block, replace the `sha256 "..."` value with the new hash.

Do not change anything else in the formula. Preserve all whitespace, comments, and structure exactly.

## Step 6 — Verify the update

After writing the file, verify it looks correct:

- Show a diff: `git diff <formula-file>`
- Confirm version string updated
- Confirm all platform URLs updated
- Confirm all SHA256 values updated
- Confirm no unintended lines changed

Ask the user to confirm the diff before committing. If anything looks wrong, ask the user how to proceed — do not attempt to auto-correct.

## Step 7 — Commit and push

```bash
git add Formula/<name>.rb
git commit -m "<name> <version-without-v>"
git push origin main
```

The commit message format matches the tap's existing convention (e.g. `claudewatch 1.2.0`, not `feat: bump claudewatch to v1.2.0`). Infer the format from recent commits in the tap: `git log --oneline -5`.

If push fails due to branch protection (bypass warning), that is expected — proceed.

## Step 8 — Report

```
Updated: <tap>/Formula/<name>.rb
Version: <old-version> → <version>
Commit:  <sha>
Push:    <status>

Platforms updated:
  darwin_arm64   <sha256>
  darwin_amd64   <sha256>
  linux_arm64    <sha256>
  linux_amd64    <sha256>
```

## Error handling policy

| Failure | Policy |
|---|---|
| Tap not found locally and clone fails | Stop, report |
| Tap working tree dirty | Stop, report |
| Tap behind remote, ff-only fails | Stop, report |
| Formula file not found | Stop, report — do not create |
| No checksum source available | Ask user to provide |
| Checksums incomplete (missing platform) | Ask user to provide missing values |
| Diff looks wrong | Ask user before committing |
| Push fails (non-bypass) | Stop, report |
| Any unexpected error | Stop, report raw output |

Do not force-push. Do not modify the formula beyond version, URL, and SHA256 fields. When in doubt, show the diff and ask.
