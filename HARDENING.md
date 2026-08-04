<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.4.0** was hardened automatically. 17 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references in build.yml use mutable tag refs instead of pinned 40-character SHA digests: `actions/checkout@v4` (line 18), `actions/setup-node@v4` (line 20), `codecov/codecov-action@v3.1.4` (line 35).

Locations:

- `.github/workflows/build.yml:18`
- `.github/workflows/build.yml:20`
- `.github/workflows/build.yml:35`

### unpinned-uses (severity: high)

All `uses:` references in codeql-analysis.yml use mutable tag refs instead of pinned 40-character SHA digests: `actions/checkout@v4` (line 21), `github/codeql-action/init@v2` (line 24), `github/codeql-action/autobuild@v2` (line 27), `github/codeql-action/analyze@v2` (line 30).

Locations:

- `.github/workflows/codeql-analysis.yml:21`
- `.github/workflows/codeql-analysis.yml:24`
- `.github/workflows/codeql-analysis.yml:27`
- `.github/workflows/codeql-analysis.yml:30`

### unpinned-uses (severity: high)

All `uses:` references in integration.yml use mutable tag refs instead of pinned 40-character SHA digests: `actions/checkout@v4` (lines 17, 40), `JamesIves/fetch-api-data-action@v2` (lines 23, 47, 57), `JamesIves/github-pages-deploy-action@v4` (lines 35, 72).

Locations:

- `.github/workflows/integration.yml:17`
- `.github/workflows/integration.yml:23`
- `.github/workflows/integration.yml:35`
- `.github/workflows/integration.yml:40`
- `.github/workflows/integration.yml:47`
- `.github/workflows/integration.yml:57`
- `.github/workflows/integration.yml:72`

### unpinned-uses (severity: high)

All `uses:` references in production.yml use mutable tag refs instead of pinned 40-character SHA digests: `actions/checkout@v4` (line 14), `actions/setup-node@v4` (line 17).

Locations:

- `.github/workflows/production.yml:14`
- `.github/workflows/production.yml:17`

### unpinned-uses (severity: high)

All `uses:` references in publish.yml use mutable tag refs instead of pinned 40-character SHA digests: `actions/checkout@v4` (line 11), `actions/setup-node@v4` (lines 16, 42).

Locations:

- `.github/workflows/publish.yml:11`
- `.github/workflows/publish.yml:16`
- `.github/workflows/publish.yml:42`

### unpinned-uses (severity: high)

All `uses:` references in sponsors.yml use mutable tag refs instead of pinned 40-character SHA digests: `actions/checkout@v4` (line 11), `JamesIves/github-sponsors-readme-action@v1` (lines 14, 23), `JamesIves/github-pages-deploy-action@v4` (line 32).

Locations:

- `.github/workflows/sponsors.yml:11`
- `.github/workflows/sponsors.yml:14`
- `.github/workflows/sponsors.yml:23`
- `.github/workflows/sponsors.yml:32`

### unpinned-uses (severity: high)

The `uses:` reference in version.yml uses a mutable tag ref instead of a pinned 40-character SHA digest: `nowactions/update-majorver@v1.1.2` (line 11).

Locations:

- `.github/workflows/version.yml:11`

### missing-permissions (severity: medium)

build.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/build.yml:1`

### missing-permissions (severity: medium)

codeql-analysis.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/codeql-analysis.yml:1`

### missing-permissions (severity: medium)

integration.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/integration.yml:1`

### missing-permissions (severity: medium)

production.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/production.yml:1`

### missing-permissions (severity: medium)

publish.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/publish.yml:1`

### missing-permissions (severity: medium)

sponsors.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/sponsors.yml:1`

### missing-permissions (severity: medium)

version.yml has no top-level `permissions:` key and no job-level `permissions:` key on any job. This means the workflow runs with the default (potentially broad) token permissions.

Locations:

- `.github/workflows/version.yml:1`

### script-injection (severity: high)

Rule (a) violation: `${{ steps.validate.outputs.fetchApiData }}` is interpolated directly inside a `run:` shell command. Step output values can contain attacker-controlled content (e.g. from API responses). Offending line: `echo "Output: ${{ steps.validate.outputs.fetchApiData }}"`

Locations:

- `.github/workflows/integration.yml:31`
- `.github/workflows/integration.yml:67`

### script-injection (severity: high)

Rule (a) violation: `${{ secrets.GIT_CONFIG_EMAIL }}`, `${{ secrets.GIT_CONFIG_NAME }}`, and `${{ github.sha }}` are interpolated directly inside a `run:` shell command. Even though secrets are trusted, any `${{ ... }}` expression inside a run: block is a script-injection risk as values flow through YAML template substitution before the shell processes them. Offending lines: `git config user.email "${{ secrets.GIT_CONFIG_EMAIL }}"`, `git config user.name "${{ secrets.GIT_CONFIG_NAME }}"`, `git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"`

Locations:

- `.github/workflows/production.yml:40`
- `.github/workflows/production.yml:41`
- `.github/workflows/production.yml:43`

### script-injection (severity: high)

Rule (a) violation: `${{ github.event.inputs.version }}` is interpolated directly inside a `run:` shell command. This is a `workflow_dispatch` input that can be supplied by any user who can trigger the workflow, enabling command injection. Offending line: `npm version ${{ github.event.inputs.version }} -m "Release ${{ github.event.inputs.version }} 📣"`

Locations:

- `.github/workflows/publish.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection

**Notes:**

Fixed all 7 workflow files:

1. **unpinned-uses**: Pinned all 10 unique action references to full 40-char SHA digests with original tags preserved as comments. Actions pinned: actions/checkout@v4, actions/setup-node@v4, codecov/codecov-action@v3.1.4, github/codeql-action/{init,autobuild,analyze}@v2, JamesIves/fetch-api-data-action@v2, JamesIves/github-pages-deploy-action@v4, JamesIves/github-sponsors-readme-action@v1, nowactions/update-majorver@v1.1.2.

2. **missing-permissions**: Added top-level `permissions:` blocks to all 7 workflows with minimal required permissions: build.yml (contents: read), codeql-analysis.yml (contents: read + security-events: write), integration.yml (contents: write), production.yml (contents: write), publish.yml (contents: write + packages: write), sponsors.yml (contents: write), version.yml (contents: write).

3. **script-injection**: Moved all ${{ }} expressions out of run: shell commands into env: blocks in integration.yml (steps.validate.outputs.fetchApiData → FETCH_API_DATA), production.yml (secrets.GIT_CONFIG_EMAIL/NAME → GIT_CONFIG_EMAIL/GIT_CONFIG_NAME, github.sha → GITHUB_SHA_VALUE), and publish.yml (github.event.inputs.version → INPUT_VERSION).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/publish.yml at the 'Authenticate with the GitHub Package Registry' step. Moved ${{ secrets.GITHUB_TOKEN }} out of the run: shell command and into an env: block as GITHUB_TOKEN_VALUE. The shell script now references it as ${GITHUB_TOKEN_VALUE} instead of directly interpolating the expression into the command string.

