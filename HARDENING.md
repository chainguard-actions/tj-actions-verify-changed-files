<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.1** was hardened automatically. 1 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step 'Glob match' uses `tj-actions/glob@v22`, which is pinned to a mutable version tag (`v22`) rather than an immutable 40-character commit SHA. This means the referenced action could be silently replaced with malicious code at any time without changing the workflow. It should be pinned to a full SHA, e.g. `tj-actions/glob@<40-char-sha> # v22`.

Locations:

- `action.yml:57`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned `tj-actions/glob@v22` to its full commit SHA `2deae40528141fc53131606d56b4e4ce2a486b29` in hardened/action/action.yml line 57. The original tag is preserved as a comment (`# v22`) for readability.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh at line 107. The CHANGED_FILES variable (which is built using INPUT_SEPARATOR from untrusted user input) was being written directly to $GITHUB_OUTPUT without sanitization. Added a sanitization step using `safe_changed_files=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')` and then writing `safe_changed_files` to $GITHUB_OUTPUT instead of the raw CHANGED_FILES. This prevents newline injection attacks that could inject additional key=value pairs into $GITHUB_OUTPUT.

