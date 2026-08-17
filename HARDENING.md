<!-- markdownlint-disable -->

# Hardening Report: cyprieng--github-breakout/v1.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **cyprieng--github-breakout/v1.2.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Two `uses:` references in .github/workflows/release.yml use mutable tag refs instead of pinned 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the referenced tags are moved or overwritten:
- `actions/checkout@v4` (line 21)
- `ncipollo/release-action@v1.19.1` (line 43)

Locations:

- `.github/workflows/release.yml:21`
- `.github/workflows/release.yml:43`

### script-injection (severity: high)

Two `run:` steps in .github/workflows/release.yml directly interpolate `${{ github.event.inputs.version }}` (a workflow_dispatch user-controlled input) into shell commands without routing through an env: variable. This allows an attacker to inject arbitrary shell commands by supplying a crafted version string.

(a) Line 27: `run: npm version --no-git-tag-version --new-version ${{ github.event.inputs.version }}`
(a) Line 32: `VERSION=${{ github.event.inputs.version }}` inside a multi-line run: block

The value is then used unquoted in subsequent shell commands (e.g., `git tag v$VERSION`, `git commit -m "$VERSION"`), compounding the risk.

Locations:

- `.github/workflows/release.yml:27`
- `.github/workflows/release.yml:32`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection

**Notes:**

1. Pinned `actions/checkout@v4` to SHA `11d5960a326750d5838078e36cf38b85af677262` and `ncipollo/release-action@v1.19.1` to SHA `1c89adf39833729d8f85a31ccbc451b078733c80`, preserving the original tags in comments.
2. Fixed script injection in the 'bump package version' step: moved `${{ github.event.inputs.version }}` into an `env:` block as `INPUT_VERSION` and referenced it as `"$INPUT_VERSION"` in the shell command.
3. Fixed script injection in the 'push new build, tag version and push' step: moved `${{ github.event.inputs.version }}` into an `env:` block as `INPUT_VERSION` and replaced the inline expression with `VERSION="$INPUT_VERSION"`. All subsequent uses of `$VERSION` are now properly double-quoted to prevent word splitting and glob expansion.
4. The `ncipollo/release-action` `with:` inputs (`tag:` and `body:`) are action input values, not shell commands, so they do not constitute script injection and were left as-is.

