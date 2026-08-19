<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in test.yml directly interpolate `${{ steps.*.outputs.changed_files }}` (a `steps.*.outputs.*` context value) into shell echo commands. This value flows through YAML template substitution before the shell processes it, enabling script injection if the output contains shell metacharacters. Offending lines include: `echo "Changed files (Not expected): ${{ steps.changed_files_not_expected.outputs.changed_files }}"`, `echo "Changed files: ${{ steps.changed_files_expected.outputs.changed_files }}"`, `echo "Changed files: ${{ steps.changed_unstaged_files_expected.outputs.changed_files }}"`, `echo "Changed files: ${{ steps.deleted_file_test.outputs.changed_files }}"`, and `echo "Deletion detected: ${{ steps.deleted_file_test.outputs.changed_files }}"`.

Locations:

- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:68`
- `.github/workflows/test.yml:112`
- `.github/workflows/test.yml:155`
- `.github/workflows/test.yml:163`

### github-env-injection (severity: high)

In entrypoint.sh, the variable `CHANGED_FILES` — derived from git-tracked filenames (which can contain newlines) and formatted using `INPUT_SEPARATOR` (sourced from `inputs.separator`, a caller-controlled input) — is written directly to `$GITHUB_OUTPUT` via `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A filename or separator containing a newline can inject arbitrary key=value pairs into the GitHub output context.

Locations:

- `entrypoint.sh:88`
- `entrypoint.sh:89`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

action.yml:
- `tj-actions/glob@v22`

.github/workflows/codacy-analysis.yml:
- `codacy/codacy-analysis-cli-action@v4.4.1`
- `github/codeql-action/upload-sarif@v3`

.github/workflows/sync-release-version.yml:
- `tj-actions/release-tagger@v4`
- `tj-actions/sync-release-version@v13`
- `tj-actions/git-cliff@v1`
- `peter-evans/create-pull-request@v6.0.5`

.github/workflows/test.yml:
- `reviewdog/action-shellcheck@v1.20`

.github/workflows/update-readme.yml:
- `tj-actions/auto-doc@v3`
- `tj-actions/remark@v3`
- `tj-actions/verify-changed-files@v20`
- `peter-evans/create-pull-request@v6.0.5`

Locations:

- `action.yml:55`
- `.github/workflows/codacy-analysis.yml:35`
- `.github/workflows/codacy-analysis.yml:48`
- `.github/workflows/sync-release-version.yml:11`
- `.github/workflows/sync-release-version.yml:13`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/sync-release-version.yml:22`
- `.github/workflows/test.yml:20`
- `.github/workflows/update-readme.yml:13`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:20`
- `.github/workflows/update-readme.yml:32`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` key, and no individual job within any of these files defines a `permissions:` key either. Without explicit permissions, workflows run with the default (potentially broad) token permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/codacy-analysis.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings:

1. script-injection: Moved all 5 `${{ steps.*.outputs.changed_files }}` interpolations in .github/workflows/test.yml into `env:` blocks, referencing them as `$CHANGED_FILES` in shell scripts.

2. github-env-injection: In entrypoint.sh, added `SAFE_CHANGED_FILES=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT to strip newlines that could inject arbitrary key=value pairs.

3. unpinned-uses: Pinned all 11 mutable tag/version references to full 40-character commit SHAs across action.yml, codacy-analysis.yml, sync-release-version.yml, test.yml, and update-readme.yml.

4. missing-permissions: Added top-level `permissions:` blocks to all 4 workflow files with minimal required permissions (codacy-analysis.yml: contents:read + security-events:write; sync-release-version.yml: contents:write + pull-requests:write; test.yml: contents:read; update-readme.yml: contents:write + pull-requests:write).

