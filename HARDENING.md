<!-- markdownlint-disable -->

# Hardening Report: cyprieng--github-breakout/v1.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cyprieng--github-breakout/v1.2.4** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The `run:` block on line 27 directly interpolates the workflow_dispatch user-controlled input `${{ github.event.inputs.version }}` into a shell command string: `npm version --no-git-tag-version --new-version ${{ github.event.inputs.version }}`. This allows an attacker to inject arbitrary shell commands via the version input.

Locations:

- `.github/workflows/release.yml:27`

### script-injection (severity: high)

Sub-rule (a) and (b): The `run:` block on line 31 directly interpolates `${{ github.event.inputs.version }}` into the shell assignment `VERSION=${{ github.event.inputs.version }}`. The resulting `$VERSION` variable is then used unquoted in `git tag v$VERSION`, `git push origin --tags --force`, and `echo $VERSION | cut ...`, violating both sub-rule (a) (expression in run: block) and sub-rule (b) (unquoted shell variable expansion of untrusted data).

Locations:

- `.github/workflows/release.yml:31`

### unpinned-uses (severity: high)

Two `uses:` references in release.yml are pinned to mutable tags rather than full 40-character commit SHA digests, making them vulnerable to supply-chain attacks: (1) `actions/checkout@v4` (line 21) — tag `v4` is mutable; (2) `ncipollo/release-action@v1.19.1` (line 44) — tag `v1.19.1` is mutable. Both should be pinned to their full SHA, e.g. `actions/checkout@<40-char-sha> # v4`.

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:44`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all three findings in hardened/action/.github/workflows/release.yml:
1. script-injection (line 27): Moved `${{ github.event.inputs.version }}` to an `env:` block as `INPUT_VERSION` and referenced it as `"$INPUT_VERSION"` in the npm version command.
2. script-injection (line 31): Moved `${{ github.event.inputs.version }}` to an `env:` block as `INPUT_VERSION`, assigned to `VERSION="$INPUT_VERSION"`, and quoted all variable expansions (`"$VERSION"`, `"v$VERSION"`, `"$VERSION"` in cut pipeline) to prevent shell injection.
3. unpinned-uses: Pinned `actions/checkout@v4` → `actions/checkout@11d5960a326750d5838078e36cf38b85af677262 # v4` and `ncipollo/release-action@v1.19.1` → `ncipollo/release-action@1c89adf39833729d8f85a31ccbc451b078733c80 # v1.19.1` using real commit SHAs.

