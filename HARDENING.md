<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action step uses `tj-actions/glob@v22`, which is a mutable tag reference rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `tj-actions/glob@<40-char-sha> # v22`.

Locations:

- `action.yml:61`

### github-env-injection (severity: high)

In entrypoint.sh, the variable `CHANGED_FILES` — which is derived from git-reported filenames (attacker-controllable via crafted filenames containing embedded newlines) and from `INPUT_SEPARATOR` (sourced from `inputs.separator`) — is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' "$CHANGED_FILES" | tr -d '\n\r'`). An attacker can create a repository file whose name contains a newline followed by `key=value` to inject arbitrary entries into GITHUB_OUTPUT, potentially hijacking downstream steps. No `tr -d` or `printf '%s'` sanitization is applied anywhere before these writes.

Locations:

- `entrypoint.sh:98`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. Pinned tj-actions/glob@v22 to full commit SHA 2deae40528141fc53131606d56b4e4ce2a486b29 in action.yml (line 61), preserving the tag as a comment. 2. In entrypoint.sh, added sanitization of CHANGED_FILES before writing to $GITHUB_OUTPUT: introduced SAFE_CHANGED_FILES=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r') and used that sanitized value in the echo statement, preventing newline injection via attacker-controlled filenames or separator values.

