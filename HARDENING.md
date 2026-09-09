<!-- markdownlint-disable -->

# Hardening Report: cpcloud--flake-dep-info-action/v2.0.11

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cpcloud--flake-dep-info-action/v2.0.11** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: GitHub Actions expressions are interpolated directly inside run: shell command strings in ci.yml. The values ${{ steps.test.outputs.owner }}, ${{ matrix.input.owner }}, ${{ steps.test.outputs.repo }}, ${{ matrix.input.repo }}, ${{ steps.test.outputs.rev }}, and ${{ steps.test.outputs.short-rev }} are all expanded by the YAML template engine before the shell ever sees them, allowing an attacker-controlled value to inject shell metacharacters. These should be moved to env: variables and double-quoted in the shell script.

Locations:

- `.github/workflows/ci.yml:52`
- `.github/workflows/ci.yml:55`
- `.github/workflows/ci.yml:58`
- `.github/workflows/ci.yml:61`

### missing-permissions (severity: medium)

Workflow files are missing a top-level `permissions:` key and none of their jobs define job-level permissions. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (often broad write access), violating the principle of least privilege. Each workflow should declare minimal required permissions at the top level or per job.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/auto-rebase.yml:1`
- `.github/workflows/update-deps.yml:1`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings rather than immutable 40-character commit SHAs. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling supply-chain attacks. Affected references:

ci.yml: `actions/checkout@v4`, `actions/setup-node@v3`, `tibdex/github-app-token@v2`, `cycjimmy/semantic-release-action@v4.0.0`

auto-rebase.yml: `tibdex/github-app-token@v2`, `Label305/AutoRebase@v0.1`

codeql-analysis.yml: `actions/checkout@v4`, `github/codeql-action/init@v2`, `github/codeql-action/autobuild@v2`, `github/codeql-action/analyze@v2`

update-deps.yml: `actions/checkout@v4` (×2), `tibdex/github-app-token@v2`, `cpcloud/flake-update-action@v1.0.4`

All of these should be pinned to a full SHA digest (e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`).

Locations:

- `.github/workflows/ci.yml:19`
- `.github/workflows/ci.yml:21`
- `.github/workflows/ci.yml:43`
- `.github/workflows/ci.yml:64`
- `.github/workflows/ci.yml:72`
- `.github/workflows/ci.yml:77`
- `.github/workflows/auto-rebase.yml:14`
- `.github/workflows/auto-rebase.yml:19`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:24`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:29`
- `.github/workflows/update-deps.yml:19`
- `.github/workflows/update-deps.yml:34`
- `.github/workflows/update-deps.yml:44`
- `.github/workflows/update-deps.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, missing-permissions, unpinned-uses

**Notes:**

Fixed all three finding types across four workflow files: (1) script-injection in ci.yml: moved all ${{ steps.test.outputs.* }} and ${{ matrix.input.* }} expressions from run: shell strings into env: blocks, referencing plain env vars in the shell; (2) missing-permissions: added top-level 'permissions: {}' to ci.yml, auto-rebase.yml, codeql-analysis.yml, and update-deps.yml, with minimal job-level permissions where needed (contents: write, pull-requests: write for release/rebase/update jobs); (3) unpinned-uses: pinned all 9 distinct action references to full 40-character commit SHAs with tag comments preserved.

