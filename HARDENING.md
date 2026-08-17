<!-- markdownlint-disable -->

# Hardening Report: cyprieng--github-breakout/v1.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cyprieng--github-breakout/v1.2.2** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct interpolation of user-controlled workflow_dispatch inputs into run: shell commands. On line 27, `${{ github.event.inputs.version }}` is interpolated directly into a shell command (`npm version --no-git-tag-version --new-version ${{ github.event.inputs.version }}`), allowing an attacker with workflow_dispatch access to inject arbitrary shell commands. On line 32, the same input is assigned to a shell variable (`VERSION=${{ github.event.inputs.version }}`) inside a run: block — the expression is expanded by the template engine before the shell sees it, enabling injection. These violate sub-rule (a): any ${{ ... }} expression directly inside a run: block is a script-injection risk.

Locations:

- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:32`

### unpinned-uses (severity: high)

Two `uses:` references in release.yml are pinned to mutable tag refs instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the referenced tags are moved or overwritten. Failing references: `actions/checkout@v4` (line 21) and `ncipollo/release-action@v1.19.1` (line 43). These should be pinned to their full SHA digests, e.g. `actions/checkout@<40-char-sha> # v4`.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:43`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed script-injection on lines 27 and 32 by moving `${{ github.event.inputs.version }}` into `env:` blocks (as INPUT_VERSION) and referencing it as `"$INPUT_VERSION"` in the shell scripts. Pinned actions/checkout@v4 to SHA 11d5960a326750d5838078e36cf38b85af677262 and ncipollo/release-action@v1.19.1 to SHA 1c89adf39833729d8f85a31ccbc451b078733c80, preserving the original tag names in comments.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in hardened/action/.github/workflows/release.yml:
1. Line 43: `git tag v$VERSION` → `git tag "v${VERSION}"`
2. Line 46: `git tag v$( echo $VERSION | cut -d. -f 1-1 )` → `git tag "v$(echo "$VERSION" | cut -d. -f 1-1)"`

Both git tag commands now properly quote the VERSION variable (which is derived from the attacker-controllable `github.event.inputs.version` input), preventing shell metacharacters from causing command injection.

