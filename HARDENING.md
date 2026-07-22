<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.4.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions using mutable version tags (e.g. @v4, @v2, @v1, @v1.1.2, @v4.6.0) instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if the tag is moved to a malicious commit.

Failing references:
- build.yml: actions/checkout@v4, actions/setup-node@v4, codecov/codecov-action@v4.6.0
- integration.yml: actions/checkout@v4 (×2), JamesIves/fetch-api-data-action@v2 (×3), JamesIves/github-pages-deploy-action@v4 (×2)
- label.yml: actions/checkout@v4, mauroalderete/action-assign-labels@v1
- production.yml: actions/checkout@v4, actions/setup-node@v4
- sponsors.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
- version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v4, actions/setup-node@v4 (×2)

Locations:

- `.github/workflows/build.yml:17`
- `.github/workflows/build.yml:20`
- `.github/workflows/build.yml:31`
- `.github/workflows/integration.yml:17`
- `.github/workflows/integration.yml:23`
- `.github/workflows/integration.yml:38`
- `.github/workflows/integration.yml:47`
- `.github/workflows/integration.yml:54`
- `.github/workflows/integration.yml:63`
- `.github/workflows/integration.yml:78`
- `.github/workflows/label.yml:22`
- `.github/workflows/label.yml:27`
- `.github/workflows/production.yml:17`
- `.github/workflows/production.yml:20`
- `.github/workflows/sponsors.yml:13`
- `.github/workflows/sponsors.yml:17`
- `.github/workflows/sponsors.yml:27`
- `.github/workflows/sponsors.yml:37`
- `.github/workflows/version.yml:13`
- `.github/workflows/version.yml:24`
- `.github/workflows/version.yml:28`
- `.github/workflows/version.yml:57`

### missing-permissions (severity: medium)

Five workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows run with the default (potentially write-all) token permissions, violating the principle of least privilege.

Affected files: build.yml, integration.yml, production.yml, sponsors.yml, version.yml.

(label.yml is the only file that correctly declares top-level permissions.)

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/production.yml:1`
- `.github/workflows/sponsors.yml:1`
- `.github/workflows/version.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions into shell commands. GitHub Actions performs this substitution before the shell parses the string, so any special characters in the value are interpreted by the shell — enabling command injection.

**integration.yml** (sub-rule a — direct expression interpolation):
- Line 35: `echo "Output: ${{ steps.validate.outputs.fetchApiData }}"` — step output value injected directly into shell
- Line 75: `echo "Output: ${{ steps.validate.outputs.fetchApiData }}"` — same pattern in second job

**production.yml** (sub-rule a):
- Line 44: `git config user.email "${{ secrets.GIT_CONFIG_EMAIL }}"` — secret value interpolated directly
- Line 45: `git config user.name "${{ secrets.GIT_CONFIG_NAME }}"` — secret value interpolated directly
- Line 47: `git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"` — github context interpolated directly

**version.yml** (sub-rule a):
- Line 62: `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc` — secret value interpolated directly into shell string

Fix: move values into `env:` variables and reference them as `$ENV_VAR` in the shell script.

Locations:

- `.github/workflows/integration.yml:35`
- `.github/workflows/integration.yml:75`
- `.github/workflows/production.yml:44`
- `.github/workflows/production.yml:45`
- `.github/workflows/production.yml:47`
- `.github/workflows/version.yml:62`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 3 findings across 6 workflow files:

1. unpinned-uses: Pinned all 22 action references to full commit SHAs:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - codecov/codecov-action@v4.6.0 → @b9fd7d16f6d7d1b5d2bec1a2887e65ceed900238
   - JamesIves/fetch-api-data-action@v2 → @8dc51e982d982157bfd575ed64be3c48b3078037
   - JamesIves/github-pages-deploy-action@v4 → @d92aa235d04922e8f08b40ce78cc5442fcfbfa2f
   - mauroalderete/action-assign-labels@v1 → @671a4ca2da0f900464c58b8b5540a1e07133e915
   - JamesIves/github-sponsors-readme-action@v1 → @2fd9142e765f755780202122261dc85e78459405
   - nowactions/update-majorver@v1.1.2 → @f2014bbbba95b635e990ce512c5653bd0f4753fb

2. missing-permissions: Added top-level permissions blocks to build.yml (contents: read), integration.yml (contents: write), production.yml (contents: write), sponsors.yml (contents: write), version.yml (contents: write + packages: write).

3. script-injection: Moved all ${{ }} expressions from run: blocks into env: blocks in integration.yml (2 instances), production.yml (3 instances), and version.yml (1 instance). Shell scripts now reference plain environment variables.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted $VERSION variable in .github/workflows/version.yml at line 47. Changed `npm version $VERSION -m "Release $VERSION 📣"` to `npm version "$VERSION" -m "Release $VERSION 📣"`. The $VERSION variable is derived from GITHUB_REF (which comes from github.ref, a workflow-controllable context), so leaving it unquoted allowed shell metacharacters in a crafted tag name to be interpreted as shell commands. Double-quoting the first positional argument prevents this injection.

