<!-- markdownlint-disable -->

# Hardening Report: cyprieng--github-breakout/v1.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cyprieng--github-breakout/v1.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ github.event.inputs.version }}` is interpolated directly inside a `run:` shell command string. An attacker who can trigger `workflow_dispatch` with a crafted version string could inject arbitrary shell commands. Offending lines:
- Line 27: `run: npm version --no-git-tag-version --new-version ${{ github.event.inputs.version }}`
- Line 31: `VERSION=${{ github.event.inputs.version }}`

Fix: move the value into an `env:` variable and double-quote it in the shell, e.g.:
```yaml
env:
  VERSION: ${{ github.event.inputs.version }}
run: npm version --no-git-tag-version --new-version "$VERSION"
```

Locations:

- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:31`

### unpinned-uses (severity: high)

Two `uses:` references are pinned to mutable tags rather than full 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the tag is moved or the upstream repository is compromised:
- Line 21: `uses: actions/checkout@v4` (tag `v4`)
- Line 41: `uses: ncipollo/release-action@v1.19.1` (tag `v1.19.1`)

Fix: pin each reference to its full SHA digest, e.g. `actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v4`.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:41`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed hardened/action/.github/workflows/release.yml:
1. script-injection: Moved `${{ github.event.inputs.version }}` out of both `run:` shell strings and into `env:` blocks as `VERSION`. The shell scripts now reference `"$VERSION"` (double-quoted) to prevent injection.
2. unpinned-uses: Pinned `actions/checkout@v4` to `@34e114876b0b11c390a56381ad16ebd13914f8d5 # v4` and `ncipollo/release-action@v1.19.1` to `@1c89adf39833729d8f85a31ccbc451b078733c80 # v1.19.1`.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the unquoted git tag command in .github/workflows/release.yml line 42. Changed `git tag v$( echo "$VERSION" | cut -d. -f 1-1 )` to `git tag "v$( echo "$VERSION" | cut -d. -f 1-1 )"`. The outer double quotes ensure the entire v<major> string is treated as a single shell word, preventing shell metacharacters in the user-controlled $VERSION input from being interpreted by the shell.

