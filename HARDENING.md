<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.5.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.5.2** was hardened automatically. 11 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ steps.validate.outputs.fetchApiData }} is directly interpolated inside a run: shell command string. Step: 'Access step output'. Offending line: `echo "Output: ${{ steps.validate.outputs.fetchApiData }}"`

Locations:

- `.github/workflows/integration.yml:48`

### script-injection (severity: high)

Rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings in the 'Commit and Push' step. Offending lines: `git config user.email "${{ secrets.GIT_CONFIG_EMAIL }}"`, `git config user.name "${{ secrets.GIT_CONFIG_NAME }}"`, `git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"`

Locations:

- `.github/workflows/production.yml:55`
- `.github/workflows/production.yml:56`
- `.github/workflows/production.yml:58`

### script-injection (severity: high)

Rule (a): Multiple ${{ }} expressions are directly interpolated inside run: shell command strings across several steps. Offending lines include: `git config user.email "${{ secrets.GIT_CONFIG_EMAIL }}"` (Configure Git), `git config user.name "${{ secrets.GIT_CONFIG_NAME }}"` (Configure Git), `NEXT_VERSION=$(npm --no-git-tag-version version "${{ inputs.bump }}")` (Compute the next version — inputs.bump is attacker-controllable via workflow_dispatch), `git checkout -b ${{ steps.version.outputs.target-branch }}` (Create the new major release branch — unquoted, rule b also violated), `git push origin ${{ steps.version.outputs.target-branch }}` (same step), `git fetch origin ${{ steps.version.outputs.target-branch }}` (Merge dev step), `git checkout ${{ steps.version.outputs.target-branch }}` (Merge dev step), `git push origin ${{ steps.version.outputs.target-branch }}` (Merge dev step), `gh release create "v${{ needs.prepare.outputs.next-version }}" --title "v${{ needs.prepare.outputs.next-version }}" --target "${{ needs.prepare.outputs.target-branch }}"` (Create the GitHub Release).

Locations:

- `.github/workflows/release.yml:33`
- `.github/workflows/release.yml:34`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:57`
- `.github/workflows/release.yml:58`
- `.github/workflows/release.yml:63`
- `.github/workflows/release.yml:64`
- `.github/workflows/release.yml:82`
- `.github/workflows/release.yml:97`

### script-injection (severity: high)

Rule (a): ${{ secrets.GITHUB_TOKEN }} is directly interpolated inside a run: shell command string in the 'Authenticate with the GitHub Package Registry' step. Offending line: `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc`

Locations:

- `.github/workflows/version.yml:62`

### unpinned-uses (severity: high)

All uses: references use mutable version tags instead of pinned 40-character SHA digests, making the workflow vulnerable to supply-chain attacks if the referenced action is compromised or its tag is moved. Unpinned references: actions/checkout@v7.0.1, actions/setup-node@v7.0.0, codecov/codecov-action@v7.0.0

Locations:

- `.github/workflows/build.yml:20`
- `.github/workflows/build.yml:23`
- `.github/workflows/build.yml:32`

### unpinned-uses (severity: high)

All uses: references use mutable version tags instead of pinned 40-character SHA digests. Unpinned references: actions/checkout@v7.0.1 (×6), JamesIves/github-pages-deploy-action@v4

Locations:

- `.github/workflows/integration.yml:27`
- `.github/workflows/integration.yml:34`
- `.github/workflows/integration.yml:55`
- `.github/workflows/integration.yml:64`
- `.github/workflows/integration.yml:68`
- `.github/workflows/integration.yml:107`
- `.github/workflows/integration.yml:111`

### unpinned-uses (severity: high)

All uses: references use mutable version tags instead of pinned 40-character SHA digests. Unpinned references: actions/checkout@v7.0.1, mauroalderete/action-assign-labels@v1.5.1

Locations:

- `.github/workflows/label.yml:14`
- `.github/workflows/label.yml:19`

### unpinned-uses (severity: high)

All uses: references use mutable version tags instead of pinned 40-character SHA digests. Unpinned references: actions/checkout@v7.0.1, actions/setup-node@v7.0.0

Locations:

- `.github/workflows/production.yml:31`
- `.github/workflows/production.yml:35`

### unpinned-uses (severity: high)

All uses: references use mutable version tags instead of pinned 40-character SHA digests. Unpinned references: actions/checkout@v7.0.1 (×2)

Locations:

- `.github/workflows/release.yml:28`
- `.github/workflows/release.yml:88`

### unpinned-uses (severity: high)

All uses: references use mutable version tags instead of pinned 40-character SHA digests. Unpinned references: actions/checkout@v7.0.1, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4

Locations:

- `.github/workflows/sponsors.yml:13`
- `.github/workflows/sponsors.yml:16`
- `.github/workflows/sponsors.yml:28`
- `.github/workflows/sponsors.yml:40`

### unpinned-uses (severity: high)

All uses: references use mutable version tags instead of pinned 40-character SHA digests. Unpinned references: nowactions/update-majorver@v1.1.2, actions/checkout@v7.0.1, actions/setup-node@v7.0.0 (×2)

Locations:

- `.github/workflows/version.yml:16`
- `.github/workflows/version.yml:31`
- `.github/workflows/version.yml:38`
- `.github/workflows/version.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses

**Notes:**

Fixed all script-injection findings by moving ${{ }} expressions from run: shell strings into env: blocks and referencing them as plain environment variables. Fixed all unpinned-uses findings by resolving each action tag to its full 40-character SHA digest using lookup_action_sha. Files modified: build.yml (pinned 3 actions), integration.yml (pinned 7 actions, fixed 1 script injection), label.yml (pinned 2 actions), production.yml (pinned 2 actions, fixed 3 script injections), release.yml (pinned 2 actions, fixed 9 script injections across 5 steps), sponsors.yml (pinned 4 actions), version.yml (pinned 4 actions, fixed 1 script injection).

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed the script injection vulnerability in .github/workflows/version.yml at line 47. Changed `npm version $VERSION -m "Release $VERSION 📣"` to `npm version "$VERSION" -m "Release $VERSION 📣"`. The VERSION variable is derived from GITHUB_REF (set via `env: GITHUB_REF: ${{ github.ref }}`), which is a workflow-controllable value. Quoting the first `$VERSION` argument prevents shell metacharacter injection from specially crafted tag names.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In .github/workflows/release.yml, the 'Compute the next version and target release branch' step now sanitizes NEXT_VERSION and NEXT_MAJOR before writing them to $GITHUB_OUTPUT. Added two sanitization lines: `SAFE_NEXT_VERSION=$(printf '%s' "$NEXT_VERSION" | tr -d '\n\r')` and `SAFE_NEXT_MAJOR=$(printf '%s' "$NEXT_MAJOR" | tr -d '\n\r')`, then replaced the raw variable references in the echo statements with the sanitized versions. This prevents any newline characters that could be injected via the npm command output from corrupting the $GITHUB_OUTPUT file format.

