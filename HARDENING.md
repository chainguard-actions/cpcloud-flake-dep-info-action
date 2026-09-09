<!-- markdownlint-disable -->

# Hardening Report: cpcloud--flake-dep-info-action/v1.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cpcloud--flake-dep-info-action/v1.1.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In ci.yml, four run: steps interpolate steps.test.outputs.* and matrix.input.* values directly into shell commands without routing through env: variables. These values flow through YAML template substitution before the shell sees them, enabling command injection. Offending lines:
- Line 47: `run: if [ "${{ steps.test.outputs.owner }}" != "${{ matrix.input.owner }}" ]; then exit 1; fi`
- Line 50: `run: if [ "${{ steps.test.outputs.repo }}" != "${{ matrix.input.repo }}" ]; then exit 1; fi`
- Line 53: `run: if [ -z "${{ steps.test.outputs.rev }}" ]; then exit 1; fi`
- Line 56: `run: if [ -z "${{ steps.test.outputs.short-rev }}" ]; then exit 1; fi`

In update-deps.yaml, a run: step interpolates ${{ matrix.input }} directly into a shell command:
- Line 27: `run: nix flake lock --update-input ${{ matrix.input }}`

Locations:

- `.github/workflows/ci.yml:47`
- `.github/workflows/ci.yml:50`
- `.github/workflows/ci.yml:53`
- `.github/workflows/ci.yml:56`
- `.github/workflows/update-deps.yaml:27`

### permissions (severity: medium)

missing-permissions: The workflow file has no top-level `permissions:` key and none of its jobs define a `permissions:` block. Without explicit permissions, the GITHUB_TOKEN is granted default (potentially broad) permissions. The codeql-analysis.yml workflow correctly defines job-level permissions, but ci.yml and update-deps.yaml do not.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/update-deps.yaml:1`

### unpinned-uses (severity: high)

Multiple `uses:` references across all three workflow files use mutable tags or branch names instead of immutable 40-character commit SHA hashes. This exposes the workflow to supply-chain attacks if the referenced action is compromised or the tag is moved.

ci.yml:
- `actions/checkout@v2`
- `actions/setup-node@v2`

codeql-analysis.yml:
- `actions/checkout@v2`
- `github/codeql-action/init@v1`
- `github/codeql-action/autobuild@v1`
- `github/codeql-action/analyze@v1`

update-deps.yaml:
- `actions/checkout@v2`
- `cachix/install-nix-action@v14`
- `cpcloud/flake-dep-info-action@main` (mutable branch)
- `cpcloud/compare-commits-action@v5.0.8`
- `peter-evans/create-pull-request@v3`
- `peter-evans/enable-pull-request-automerge@v1`

Locations:

- `.github/workflows/ci.yml:14`
- `.github/workflows/ci.yml:18`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:31`
- `.github/workflows/codeql-analysis.yml:34`
- `.github/workflows/update-deps.yaml:14`
- `.github/workflows/update-deps.yaml:18`
- `.github/workflows/update-deps.yaml:26`
- `.github/workflows/update-deps.yaml:33`
- `.github/workflows/update-deps.yaml:42`
- `.github/workflows/update-deps.yaml:52`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, permissions, unpinned-uses

**Notes:**

Fixed all three findings across ci.yml, update-deps.yaml, and codeql-analysis.yml:

1. script-injection: Moved all ${{ }} expressions in run: steps to env: blocks in ci.yml (4 steps: check owner, check repo, check rev, check short-rev) and update-deps.yaml (nix flake lock step now uses MATRIX_INPUT env var).

2. permissions: Added `permissions: {}` top-level block to ci.yml and update-deps.yaml. codeql-analysis.yml already had job-level permissions.

3. unpinned-uses: Pinned all 10 action references to their full 40-character commit SHAs with original tags preserved as comments: actions/checkout@v2, actions/setup-node@v2, github/codeql-action/{init,autobuild,analyze}@v1, cachix/install-nix-action@v14, cpcloud/flake-dep-info-action@main, cpcloud/compare-commits-action@v5.0.8, peter-evans/create-pull-request@v3, peter-evans/enable-pull-request-automerge@v1.

