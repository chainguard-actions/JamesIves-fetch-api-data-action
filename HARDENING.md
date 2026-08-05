<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.4.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.4.1** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference actions using mutable tags or version strings instead of pinned 40-character commit SHAs. This exposes the workflow to supply-chain attacks if an action's tag is moved to a malicious commit.

build.yml: actions/checkout@v4, actions/setup-node@v4, codecov/codecov-action@v4.3.0
integration.yml: actions/checkout@v4, JamesIves/fetch-api-data-action@v2 (×2), JamesIves/github-pages-deploy-action@v4 (×2)
label.yml: actions/checkout@v3, mauroalderete/action-assign-labels@v1
production.yml: actions/checkout@v4, actions/setup-node@v4
sponsors.yml: actions/checkout@v4, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v4, actions/setup-node@v4 (×2)

Locations:

- `.github/workflows/build.yml:16`
- `.github/workflows/build.yml:18`
- `.github/workflows/build.yml:29`
- `.github/workflows/integration.yml:20`
- `.github/workflows/integration.yml:25`
- `.github/workflows/integration.yml:41`
- `.github/workflows/integration.yml:50`
- `.github/workflows/integration.yml:55`
- `.github/workflows/integration.yml:63`
- `.github/workflows/integration.yml:79`
- `.github/workflows/label.yml:14`
- `.github/workflows/label.yml:18`
- `.github/workflows/production.yml:17`
- `.github/workflows/production.yml:19`
- `.github/workflows/sponsors.yml:12`
- `.github/workflows/sponsors.yml:15`
- `.github/workflows/sponsors.yml:24`
- `.github/workflows/sponsors.yml:33`
- `.github/workflows/version.yml:12`
- `.github/workflows/version.yml:18`
- `.github/workflows/version.yml:22`
- `.github/workflows/version.yml:46`

### script-injection (severity: high)

Multiple run: blocks directly interpolate GitHub Actions expressions (${{ ... }}) inside shell commands, allowing template substitution before the shell parses the string.

(a) integration.yml — two steps echo ${{ steps.validate.outputs.fetchApiData }} directly in a run: block. A malicious API response stored in that output could inject shell metacharacters.
  Line ~37: echo "Output: ${{ steps.validate.outputs.fetchApiData }}"
  Line ~72: echo "Output: ${{ steps.validate.outputs.fetchApiData }}"

(a) production.yml — ${{ github.sha }} is interpolated directly into a git commit message inside a run: block:
  Line ~40: git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"

(a) version.yml — ${{ secrets.GITHUB_TOKEN }} is interpolated directly into an echo command inside a run: block:
  Line ~50: echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc

(b) version.yml — $VERSION (derived from the env var GITHUB_REF=${{ github.ref }}) is used unquoted in a shell command, allowing word-splitting and glob expansion on the tag value:
  Line ~43: npm version $VERSION -m "Release $VERSION 📣"

Locations:

- `.github/workflows/integration.yml:37`
- `.github/workflows/integration.yml:72`
- `.github/workflows/production.yml:40`
- `.github/workflows/version.yml:43`
- `.github/workflows/version.yml:50`

### missing-permissions (severity: medium)

Five workflow files have no top-level permissions: key and no per-job permissions: keys. Without explicit permissions, workflows run with the default token permissions (which may be read/write depending on repository settings), violating the principle of least privilege. Only label.yml defines explicit permissions.

Affected files: build.yml, integration.yml, production.yml, sponsors.yml, version.yml

Locations:

- `.github/workflows/build.yml:1`
- `.github/workflows/integration.yml:1`
- `.github/workflows/production.yml:1`
- `.github/workflows/sponsors.yml:1`
- `.github/workflows/version.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types across 6 workflow files:

1. unpinned-uses: Pinned all 22 action references to full 40-char commit SHAs with original tag as comment:
   - actions/checkout@v4 → @11d5960a326750d5838078e36cf38b85af677262
   - actions/checkout@v3 → @a37ce9120846195fa4ece8f58b268e6043cb2f26
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - codecov/codecov-action@v4.3.0 → @84508663e988701840491b86de86b666e8a86bed
   - JamesIves/fetch-api-data-action@v2 → @8dc51e982d982157bfd575ed64be3c48b3078037
   - JamesIves/github-pages-deploy-action@v4 → @d92aa235d04922e8f08b40ce78cc5442fcfbfa2f
   - JamesIves/github-sponsors-readme-action@v1 → @2fd9142e765f755780202122261dc85e78459405
   - mauroalderete/action-assign-labels@v1 → @671a4ca2da0f900464c58b8b5540a1e07133e915
   - nowactions/update-majorver@v1.1.2 → @f2014bbbba95b635e990ce512c5653bd0f4753fb

2. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks:
   - integration.yml (×2): steps.validate.outputs.fetchApiData → env FETCH_API_DATA
   - production.yml: github.sha → env COMMIT_SHA; secrets.GIT_CONFIG_EMAIL/NAME → env vars
   - version.yml: secrets.GITHUB_TOKEN in echo → env GITHUB_TOKEN; quoted $VERSION in npm version command

3. missing-permissions: Added top-level permissions blocks to build.yml (contents: read), integration.yml (contents: write), production.yml (contents: write), sponsors.yml (contents: write), version.yml (contents: write, packages: write)

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed unquoted shell variable expansion in `.github/workflows/version.yml` line 40. Changed `VERSION=${GITHUB_REF#refs/tags/v}` to `VERSION="${GITHUB_REF#refs/tags/v}"`. The `GITHUB_REF` env var was already properly isolated in the step's `env:` block (not inlined as a `${{ }}` expression in the run script), so only the quoting of the bash parameter substitution needed to be added to prevent shell metacharacter injection from a crafted tag name.

