<!-- markdownlint-disable -->

# Hardening Report: JamesIves--fetch-api-data-action/v2.5.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **JamesIves--fetch-api-data-action/v2.5.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files is pinned to a mutable tag or version string rather than a full 40-character SHA commit hash. This exposes the workflows to supply-chain attacks if any referenced action's tag is moved or compromised.

Failing references include:
- build.yml: actions/checkout@v7.0.1, actions/setup-node@v7.0.0, codecov/codecov-action@v7.0.0
- integration.yml: actions/checkout@v7.0.1 (×5), JamesIves/github-pages-deploy-action@v4
- label.yml: actions/checkout@v7.0.1, mauroalderete/action-assign-labels@v1.5.1
- production.yml: actions/checkout@v7.0.1, actions/setup-node@v7.0.0
- release.yml: actions/checkout@v7.0.1 (×2)
- sponsors.yml: actions/checkout@v7.0.1, JamesIves/github-sponsors-readme-action@v1 (×2), JamesIves/github-pages-deploy-action@v4
- version.yml: nowactions/update-majorver@v1.1.2, actions/checkout@v7.0.1, actions/setup-node@v7.0.0 (×2)

Locations:

- `.github/workflows/build.yml:22`
- `.github/workflows/build.yml:24`
- `.github/workflows/build.yml:34`
- `.github/workflows/integration.yml:28`
- `.github/workflows/integration.yml:33`
- `.github/workflows/integration.yml:57`
- `.github/workflows/label.yml:17`
- `.github/workflows/label.yml:21`
- `.github/workflows/production.yml:34`
- `.github/workflows/production.yml:38`
- `.github/workflows/release.yml:29`
- `.github/workflows/sponsors.yml:15`
- `.github/workflows/sponsors.yml:18`
- `.github/workflows/sponsors.yml:27`
- `.github/workflows/sponsors.yml:35`
- `.github/workflows/version.yml:18`
- `.github/workflows/version.yml:35`
- `.github/workflows/version.yml:40`
- `.github/workflows/version.yml:69`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ ... }}` expressions, which are expanded by the GitHub Actions template engine before the shell ever sees the string. This allows injection of arbitrary shell commands.

**Sub-rule (a) — direct expression interpolation in run blocks:**

1. integration.yml line 55: `echo "Output: ${{ steps.validate.outputs.fetchApiData }}"` — steps.*.outputs.* value interpolated directly into shell command.

2. release.yml line 44: `NEXT_VERSION=$(npm --no-git-tag-version version "${{ inputs.bump }}")` — attacker-controlled `inputs.bump` (workflow_dispatch choice input) interpolated directly into shell command.

3. release.yml line 65–66: `git checkout -b ${{ steps.version.outputs.target-branch }}` and `git push origin ${{ steps.version.outputs.target-branch }}` — steps.*.outputs.* interpolated directly and unquoted.

4. release.yml line 71–72: `git fetch origin ${{ steps.version.outputs.target-branch }}` and `git checkout ${{ steps.version.outputs.target-branch }}` — same.

5. release.yml (~line 90): `git push origin ${{ steps.version.outputs.target-branch }}` — same.

6. release.yml (~line 108–110): `gh release create "v${{ needs.prepare.outputs.next-version }}"`, `--title "v${{ needs.prepare.outputs.next-version }}"`, `--target "${{ needs.prepare.outputs.target-branch }}"` — needs.*.outputs.* interpolated directly.

7. production.yml line 62–63: `git config user.email "${{ secrets.GIT_CONFIG_EMAIL }}"` and `git config user.name "${{ secrets.GIT_CONFIG_NAME }}"` — expressions directly in run.

8. production.yml line 65: `git commit -m "Deploy Production Code for Commit ${{ github.sha }} 🚀"` — github.* directly in run.

9. release.yml line 36–37: `git config user.email "${{ secrets.GIT_CONFIG_EMAIL }}"` and `git config user.name "${{ secrets.GIT_CONFIG_NAME }}"` — expressions directly in run.

10. version.yml line 77: `echo "//npm.pkg.github.com:_authToken=${{ secrets.GITHUB_TOKEN }}" > ~/.npmrc` — expression directly in run.

**Sub-rule (b) — unquoted shell variable expansion of workflow-controlled data:**

11. version.yml line 58: `npm version $VERSION -m "Release $VERSION 📣"` — `$VERSION` is derived from `GITHUB_REF` (set from `${{ github.ref }}`) and is unquoted, allowing shell metacharacter injection.

Locations:

- `.github/workflows/integration.yml:55`
- `.github/workflows/release.yml:44`
- `.github/workflows/release.yml:65`
- `.github/workflows/release.yml:66`
- `.github/workflows/release.yml:71`
- `.github/workflows/release.yml:72`
- `.github/workflows/release.yml:36`
- `.github/workflows/release.yml:108`
- `.github/workflows/production.yml:62`
- `.github/workflows/production.yml:65`
- `.github/workflows/version.yml:58`
- `.github/workflows/version.yml:77`

### github-env-injection (severity: high)

In release.yml, the 'Compute the next version and target release branch' step writes values derived from `inputs.bump` (an attacker-controllable `workflow_dispatch` input) to `$GITHUB_OUTPUT` without sanitization.

The value `NEXT_VERSION` is computed as:
```
NEXT_VERSION=$(npm --no-git-tag-version version "${{ inputs.bump }}")
```
and then written unsanitized:
```
echo "next-version=$NEXT_VERSION" >> "$GITHUB_OUTPUT"
echo "target-branch=releases/v$NEXT_MAJOR" >> "$GITHUB_OUTPUT"
```

If `inputs.bump` contains newline characters (e.g., via a crafted value), the write to `$GITHUB_OUTPUT` could inject additional key=value pairs, allowing an attacker to poison subsequent steps' environment. The required sanitization step (`printf '%s' "$NEXT_VERSION" | tr -d '\n\r'`) is absent before each write.

Locations:

- `.github/workflows/release.yml:54`
- `.github/workflows/release.yml:55`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings across 6 workflow files:

1. unpinned-uses: Pinned all 7 distinct action references to full 40-char SHAs with original tag in comment. Affected files: build.yml, integration.yml, label.yml, production.yml, release.yml, sponsors.yml, version.yml.

2. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks. Key fixes: integration.yml (steps output), release.yml (inputs.bump, steps outputs, needs outputs, secrets), production.yml (secrets, github.sha), version.yml (secrets.GITHUB_TOKEN, fixed unquoted $VERSION to use "$VERSION").

3. github-env-injection: In release.yml, sanitized NEXT_VERSION and NEXT_MAJOR with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection.

