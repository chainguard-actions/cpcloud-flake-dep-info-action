<!-- markdownlint-disable -->

# Hardening Report: cpcloud--flake-dep-info-action/v2.0.13

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cpcloud--flake-dep-info-action/v2.0.13** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions by mutable tags or version strings instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

In .github/workflows/ci.yml:
  - uses: actions/checkout@v4
  - uses: actions/setup-node@v4
  - uses: actions/setup-node@v3
  - uses: cycjimmy/semantic-release-action@v4.1.1 (appears twice)
  - uses: actions/create-github-app-token@v1.11.0

In .github/workflows/auto-rebase.yml:
  - uses: actions/create-github-app-token@v1.11.0
  - uses: Label305/AutoRebase@v0.1

In .github/workflows/codeql-analysis.yml:
  - uses: actions/checkout@v4
  - uses: github/codeql-action/init@v3
  - uses: github/codeql-action/autobuild@v3
  - uses: github/codeql-action/analyze@v3

In .github/workflows/update-deps.yml:
  - uses: actions/checkout@v4 (appears twice)
  - uses: actions/create-github-app-token@v1.11.0
  - uses: cpcloud/flake-update-action@v2.0.1

Note: cachix/install-nix-action@08dcb3a5e62fa31e2da3d490afc4176ef55ecd72 is correctly pinned.

Locations:

- `.github/workflows/ci.yml:18`
- `.github/workflows/ci.yml:20`
- `.github/workflows/ci.yml:63`
- `.github/workflows/ci.yml:70`
- `.github/workflows/ci.yml:79`
- `.github/workflows/ci.yml:88`
- `.github/workflows/ci.yml:99`
- `.github/workflows/auto-rebase.yml:14`
- `.github/workflows/auto-rebase.yml:19`
- `.github/workflows/codeql-analysis.yml:20`
- `.github/workflows/codeql-analysis.yml:22`
- `.github/workflows/codeql-analysis.yml:26`
- `.github/workflows/codeql-analysis.yml:28`
- `.github/workflows/update-deps.yml:18`
- `.github/workflows/update-deps.yml:38`
- `.github/workflows/update-deps.yml:44`
- `.github/workflows/update-deps.yml:50`

### script-injection (severity: high)

ci.yml contains four run: steps that directly interpolate GitHub Actions expressions (${{ ... }}) inside shell command strings (sub-rule a). The expressions include steps.*.outputs.* and matrix.* values, which are workflow-controllable and flow through YAML template substitution before the shell sees them. An attacker who can influence matrix values or step outputs could inject arbitrary shell commands.

Offending lines:
  Line 51: run: if [ "${{ steps.test.outputs.owner }}" != "${{ matrix.input.owner }}" ]; then exit 1; fi
  Line 54: run: if [ "${{ steps.test.outputs.repo }}" != "${{ matrix.input.repo }}" ]; then exit 1; fi
  Line 57: run: if [ -z "${{ steps.test.outputs.rev }}" ]; then exit 1; fi
  Line 60: run: if [ -z "${{ steps.test.outputs.short-rev }}" ]; then exit 1; fi

Fix: move the values into env: variables and reference them as quoted shell variables (e.g. "$OWNER") instead of using ${{ }} directly in the run: block.

Locations:

- `.github/workflows/ci.yml:51`
- `.github/workflows/ci.yml:54`
- `.github/workflows/ci.yml:57`
- `.github/workflows/ci.yml:60`

### missing-permissions (severity: medium)

Three workflow files have no top-level permissions: key and no job-level permissions: key on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be read/write), violating the principle of least privilege.

- .github/workflows/ci.yml: jobs build, test, dry-release, and release all lack permissions.
- .github/workflows/auto-rebase.yml: job auto-rebase lacks permissions.
- .github/workflows/update-deps.yml: jobs get-flakes and flake-update lack permissions.

.github/workflows/codeql-analysis.yml correctly defines job-level permissions on its analyze job.

Locations:

- `.github/workflows/ci.yml:1`
- `.github/workflows/auto-rebase.yml:1`
- `.github/workflows/update-deps.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all findings across four workflow files:

**unpinned-uses**: Pinned all action references to full commit SHAs with tag comments:
- actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
- actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
- actions/setup-node@v3 → @3235b876344d2a9aa001b8d1453c930bba69e610
- cycjimmy/semantic-release-action@v4.1.1 → @b1b432f13acb7768e0c8efdec416d363a57546f2
- actions/create-github-app-token@v1.11.0 → @5d869da34e18e7287c1daad50e0b8ea0f506ce69
- Label305/AutoRebase@v0.1 → @e2bf9fd616286d8feddcaba773a8f2ea9011c745
- github/codeql-action/init@v3 → @6f5948dfacef28e207b48d0905cf90c03365536d
- github/codeql-action/autobuild@v3 → @6f5948dfacef28e207b48d0905cf90c03365536d
- github/codeql-action/analyze@v3 → @6f5948dfacef28e207b48d0905cf90c03365536d
- cpcloud/flake-update-action@v2.0.1 → @10ccab3efc5659d7562dd2368990e081f3ee7ac9

**script-injection**: Moved all four ${{ }} expressions in ci.yml run: steps into env: blocks (TEST_OWNER, MATRIX_OWNER, TEST_REPO, MATRIX_REPO, TEST_REV, TEST_SHORT_REV) and referenced them as plain shell variables.

**missing-permissions**: Added minimal permissions to all jobs:
- ci.yml build/test/dry-release: contents: read
- ci.yml release: contents: write, pull-requests: write (needed for semantic-release)
- auto-rebase.yml auto-rebase: contents: write, pull-requests: write (needed for rebasing)
- update-deps.yml get-flakes: contents: read
- update-deps.yml flake-update: contents: write, pull-requests: write (needed to create PRs)

