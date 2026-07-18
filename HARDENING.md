<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.2** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files and action.yml reference external actions using mutable tags or version strings instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks.

action.yml: `tj-actions/glob@v22`

.github/workflows/test.yml: `reviewdog/action-shellcheck@v1.29`

.github/workflows/codacy-analysis.yml: `codacy/codacy-analysis-cli-action@v4.4.5`, `github/codeql-action/upload-sarif@v3`

.github/workflows/sync-release-version.yml: `tj-actions/release-tagger@v4`, `tj-actions/sync-release-version@v13`, `tj-actions/git-cliff@v1`, `peter-evans/create-pull-request@v7.0.8`

.github/workflows/update-readme.yml: `tj-actions/auto-doc@v3`, `tj-actions/remark@v3`, `tj-actions/verify-changed-files@v20`, `peter-evans/create-pull-request@v7.0.8`

Locations:

- `action.yml:55`
- `.github/workflows/test.yml:19`
- `.github/workflows/codacy-analysis.yml:36`
- `.github/workflows/codacy-analysis.yml:50`
- `.github/workflows/sync-release-version.yml:12`
- `.github/workflows/sync-release-version.yml:15`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/sync-release-version.yml:22`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:20`
- `.github/workflows/update-readme.yml:33`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` block, and no individual job within them defines a `permissions:` block either. Without explicit permissions, workflows run with the default (potentially broad) GITHUB_TOKEN permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/test.yml:1`
- `.github/workflows/codacy-analysis.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/update-readme.yml:1`

### github-env-injection (severity: high)

In entrypoint.sh, the variable `CHANGED_FILES` is written directly to `$GITHUB_OUTPUT` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). `CHANGED_FILES` is constructed using `INPUT_SEPARATOR` as a delimiter, which is set from `inputs.separator` by the calling workflow (an untrusted, workflow-controlled value). A malicious caller could inject newlines into `INPUT_SEPARATOR` to smuggle additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other output values.

Offending line: `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"`

Locations:

- `entrypoint.sh:88`

### script-injection (severity: high)

Multiple `run:` blocks in .github/workflows/test.yml directly interpolate `${{ steps.*.outputs.changed_files }}` expressions inside shell commands. The `steps.*.outputs.*` context is workflow-controllable and flows through YAML template substitution before the shell processes it, allowing an attacker to inject shell metacharacters. Sub-rule (a): direct expression interpolation in run: blocks.

Offending lines:
- `echo "Changed files (Not expected): ${{ steps.changed_files_not_expected.outputs.changed_files }}"`
- `echo "Changed files: ${{ steps.changed_files_expected.outputs.changed_files }}"`
- `echo "Changed files: ${{ steps.changed_unstaged_files_expected.outputs.changed_files }}"`
- `echo "Changed files: ${{ steps.deleted_file_test.outputs.changed_files }}"`
- `echo "Deletion detected: ${{ steps.deleted_file_test.outputs.changed_files }}"`

Locations:

- `.github/workflows/test.yml:36`
- `.github/workflows/test.yml:72`
- `.github/workflows/test.yml:113`
- `.github/workflows/test.yml:152`
- `.github/workflows/test.yml:158`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, github-env-injection, script-injection

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned all 12 unpinned action references to full 40-char SHAs with original tags as comments. Actions pinned: tj-actions/glob@v22, reviewdog/action-shellcheck@v1.29, codacy/codacy-analysis-cli-action@v4.4.5, github/codeql-action/upload-sarif@v3, tj-actions/release-tagger@v4, tj-actions/sync-release-version@v13, tj-actions/git-cliff@v1, peter-evans/create-pull-request@v7.0.8 (x2), tj-actions/auto-doc@v3, tj-actions/remark@v3, tj-actions/verify-changed-files@v20.

2. missing-permissions: Added top-level permissions blocks to all four workflow files with minimal required permissions (contents:read for test.yml, contents:read+security-events:write for codacy-analysis.yml, contents:write+pull-requests:write for sync-release-version.yml and update-readme.yml).

3. github-env-injection: Fixed entrypoint.sh line 88 by sanitizing CHANGED_FILES with `printf '%s' "$CHANGED_FILES" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT.

4. script-injection: Fixed all 5 offending run: blocks in .github/workflows/test.yml by moving ${{ steps.*.outputs.changed_files }} expressions into step-level env: blocks and referencing them as $CHANGED_FILES in the shell scripts.

