<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.5.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across all workflow files use mutable tag/version strings instead of immutable 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved.

build.yml: actions/checkout@v6.0.1, actions/setup-node@v6.2.0, codecov/codecov-action@v5.5.2
integration.yml: actions/checkout@v6.0.1, JamesIves/fetch-api-data-action@v2, JamesIves/github-pages-deploy-action@v4
label.yml: actions/checkout@v6.0.1, mauroalderete/action-assign-labels@v1.5.1
production.yml: actions/checkout@v6.0.1, actions/setup-node@v6.2.0
sponsors.yml: actions/checkout@v6.0.1, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v6.0.1, actions/setup-node@v6.2.0 (×2)

Locations:

- `.github/workflows/build.yml:17`
- `.github/workflows/build.yml:19`
- `.github/workflows/build.yml:33`
- `.github/workflows/integration.yml:17`
- `.github/workflows/integration.yml:23`
- `.github/workflows/integration.yml:37`
- `.github/workflows/label.yml:18`
- `.github/workflows/label.yml:23`
- `.github/workflows/production.yml:17`
- `.github/workflows/production.yml:19`
- `.github/workflows/sponsors.yml:13`
- `.github/workflows/sponsors.yml:16`
- `.github/workflows/sponsors.yml:24`
- `.github/workflows/sponsors.yml:33`
- `.github/workflows/version.yml:13`
- `.github/workflows/version.yml:25`
- `.github/workflows/version.yml:31`
- `.github/workflows/version.yml:57`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` key and no job-level `permissions:` keys on any of their jobs. Without explicit permissions, workflows run with the default (often write-all) token permissions, granting unnecessary access. Only label.yml has explicit permissions defined.

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/production.yml:1`
- `.github/workflows/sponsors.yml:1`
- `.github/workflows/version.yml:1`

### script-injection (severity: high)

GitHub Actions expressions (`${{ ... }}`) are interpolated directly inside `run:` shell command strings (sub-rule a), allowing the expression value to be parsed by the shell before any quoting takes effect.

1. integration.yml line 36: `echo "Output: ${{ steps.validate.outputs.fetchApiData }}"` — step output expression directly in run block; a malicious API response stored in the step output could inject shell commands.

2. production.yml line 44: `git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"` — github.sha expression directly in run block.

3. version.yml line 63: `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc` — secrets expression directly in run block; the secret value is expanded by the YAML template engine before the shell sees it.

Locations:

- `.github/workflows/integration.yml:36`
- `.github/workflows/production.yml:44`
- `.github/workflows/version.yml:63`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all findings across 6 workflow files:

**unpinned-uses**: Pinned all 18 `uses:` references to full 40-char SHA hashes with original tags as comments:
- actions/checkout@v6.0.1 → @8e8c483db84b4bee98b60c0593521ed34d9990e8
- actions/setup-node@v6.2.0 → @6044e13b5dc448c55e2357c09f80417699197238
- codecov/codecov-action@v5.5.2 → @671740ac38dd9b0130fbe1cec585b89eea48d3de
- JamesIves/fetch-api-data-action@v2 → @8dc51e982d982157bfd575ed64be3c48b3078037
- JamesIves/github-pages-deploy-action@v4 → @d92aa235d04922e8f08b40ce78cc5442fcfbfa2f
- mauroalderete/action-assign-labels@v1.5.1 → @671a4ca2da0f900464c58b8b5540a1e07133e915
- JamesIves/github-sponsors-readme-action@v1 → @2fd9142e765f755780202122261dc85e78459405
- nowactions/update-majorver@v1.1.2 → @f2014bbbba95b635e990ce512c5653bd0f4753fb

**missing-permissions**: Added top-level `permissions:` blocks to build.yml (contents: read), integration.yml (contents: write), production.yml (contents: write), sponsors.yml (contents: write), and version.yml (contents: write).

**script-injection**: Fixed 3 locations by moving `${{ }}` expressions into `env:` blocks and referencing them as plain env vars in shell scripts: integration.yml (steps.validate.outputs.fetchApiData), production.yml (github.sha + secrets), version.yml (secrets.GITHUB_TOKEN).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in hardened/action/.github/workflows/version.yml at line 54. Changed `npm version $VERSION -m "Release $VERSION 📣"` to `npm version "$VERSION" -m "Release $VERSION 📣"`. The $VERSION variable (derived from the attacker-controlled github.ref tag value) was being expanded unquoted as the first argument to npm version, allowing shell metacharacters in a crafted tag name to achieve command injection. Double-quoting the variable prevents word splitting and glob expansion.

