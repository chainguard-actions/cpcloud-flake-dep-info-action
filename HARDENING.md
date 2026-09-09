<!-- markdownlint-disable -->

# Hardening Report: cpcloud--flake-dep-info-action/v2.0.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cpcloud--flake-dep-info-action/v2.0.10** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag-based `uses:` references instead of pinned 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if the referenced tags are moved or compromised.

**.github/workflows/ci.yml** failing references:
- `actions/checkout@v3`
- `actions/setup-node@v3`
- `tibdex/github-app-token@v1`
- `cycjimmy/semantic-release-action@v2.7.0`

**.github/workflows/auto-rebase.yml** failing references:
- `tibdex/github-app-token@v1`
- `Label305/AutoRebase@v0.1`

**.github/workflows/codeql-analysis.yml** failing references:
- `actions/checkout@v3`
- `github/codeql-action/init@v2`
- `github/codeql-action/autobuild@v2`
- `github/codeql-action/analyze@v2`

**.github/workflows/update-deps.yml** failing references:
- `actions/checkout@v3`
- `cachix/install-nix-action@v18`
- `tibdex/github-app-token@v1`
- `cpcloud/flake-update-action@v1.0.2`

Locations:

- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:40`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:68`
- `.github/workflows/auto-rebase.yml:14`
- `.github/workflows/auto-rebase.yml:19`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:24`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:28`
- `.github/workflows/update-deps.yml:17`
- `.github/workflows/update-deps.yml:18`
- `.github/workflows/update-deps.yml:33`
- `.github/workflows/update-deps.yml:38`
- `.github/workflows/update-deps.yml:41`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, workflows inherit the default (often broad) repository permissions, violating the principle of least privilege.

- **.github/workflows/ci.yml**: Jobs `build`, `test`, and `release` all lack `permissions:` blocks, and there is no top-level `permissions:` key.
- **.github/workflows/auto-rebase.yml**: Job `auto-rebase` lacks a `permissions:` block, and there is no top-level `permissions:` key.
- **.github/workflows/update-deps.yml**: Jobs `get-flakes` and `flake-update` both lack `permissions:` blocks, and there is no top-level `permissions:` key.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/auto-rebase.yml:1`
- `.github/workflows/update-deps.yml:1`

### script-injection (severity: high)

Four `run:` steps in `.github/workflows/ci.yml` directly interpolate GitHub Actions expressions (`${{ ... }}`) inside shell command strings (sub-rule a). Before the shell executes the command, GitHub Actions substitutes the expression value as raw text, allowing an attacker-controlled value to inject arbitrary shell commands.

The affected steps interpolate `steps.test.outputs.*` (values derived from the action under test, which reads from a lockfile) and `matrix.input.*` (values from the workflow matrix). While the matrix values are defined in the workflow itself and are low-risk, `steps.test.outputs.*` values flow from the action's output and should not be interpolated directly.

Offending lines:
- Line 47: `run: if [ "${{ steps.test.outputs.owner }}" != "${{ matrix.input.owner }}" ]; then exit 1; fi`
- Line 50: `run: if [ "${{ steps.test.outputs.repo }}" != "${{ matrix.input.repo }}" ]; then exit 1; fi`
- Line 53: `run: if [ -z "${{ steps.test.outputs.rev }}" ]; then exit 1; fi`
- Line 56: `run: if [ -z "${{ steps.test.outputs.short-rev }}" ]; then exit 1; fi`

Fix: Move the values into `env:` variables and reference them as quoted shell variables (e.g., `"$STEP_OUTPUT"`) instead of using `${{ }}` directly in the `run:` block.

Locations:

- `.github/workflows/ci.yml:47`
- `.github/workflows/ci.yml:50`
- `.github/workflows/ci.yml:53`
- `.github/workflows/ci.yml:56`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across four workflow files:

**unpinned-uses**: Pinned all 16 mutable tag references to full 40-char commit SHAs:
- actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26
- actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610
- tibdex/github-app-token@v1 → @32691ba7c9e7063bd457bd8f2a5703138591fa58
- cycjimmy/semantic-release-action@v2.7.0 → @5982a02995853159735cb838992248c4f0f16166
- Label305/AutoRebase@v0.1 → @e2bf9fd616286d8feddcaba773a8f2ea9011c745
- github/codeql-action/init@v2 → @b8d3b6e8af63cde30bdc382c0bc28114f4346c88
- github/codeql-action/autobuild@v2 → @b8d3b6e8af63cde30bdc382c0bc28114f4346c88
- github/codeql-action/analyze@v2 → @b8d3b6e8af63cde30bdc382c0bc28114f4346c88
- cachix/install-nix-action@v18 → @daddc62a2e67d1decb56e028c9fa68344b9b7c2a
- cpcloud/flake-update-action@v1.0.2 → @eea7b63ddc8fb5c546064ea6ad485a414e259a76

**missing-permissions**: Added `permissions: {}` at top level of ci.yml, auto-rebase.yml, update-deps.yml, and codeql-analysis.yml. Added minimal job-level permissions (contents:read for build/test/get-flakes, contents:write+pull-requests:write for release/auto-rebase/flake-update).

**script-injection**: Fixed 4 run steps in ci.yml by moving ${{ steps.test.outputs.* }} and ${{ matrix.input.* }} expressions into env: blocks and referencing them as plain shell variables ($STEP_OWNER, $MATRIX_OWNER, $STEP_REPO, $MATRIX_REPO, $STEP_REV, $STEP_SHORT_REV).

