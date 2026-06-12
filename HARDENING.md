<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tj-actions--verify-changed-files/v20.0.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable CHANGED_FILES is constructed using INPUT_SEPARATOR (which is sourced from inputs.separator via the env: block in action.yml) and then written directly to $GITHUB_OUTPUT without sanitization: `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"`. The INPUT_SEPARATOR value flows from the calling workflow's inputs.separator into the awk join command that builds CHANGED_FILES, and the result is written to GITHUB_OUTPUT without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. A malicious caller could supply a separator containing newline characters to inject arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `entrypoint.sh:88`
- `action.yml:68`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in entrypoint.sh at line 88. The CHANGED_FILES variable was being written directly to $GITHUB_OUTPUT without sanitization. A malicious caller could supply a separator (inputs.separator → INPUT_SEPARATOR) containing newline characters, which would be incorporated into CHANGED_FILES via the awk join command, and then injected into GITHUB_OUTPUT as arbitrary key=value pairs. The fix adds a sanitization step: `safe_changed_files=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')` before writing to GITHUB_OUTPUT, ensuring any embedded newlines or carriage returns are stripped.

