<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable CHANGED_FILES — which is derived from INPUT_SEPARATOR (an inherited env var set from inputs.separator, a caller-controlled input) and from git-tracked filenames — is written directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). The line `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"` can be exploited: if INPUT_SEPARATOR contains an actual newline byte (the percent-encoding substitutions at the top of the script only handle literal \n/\r escape sequences, not real newline bytes), an attacker-controlled calling workflow could inject additional key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent output values or poisoning downstream steps.

Locations:

- `entrypoint.sh:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed entrypoint.sh line 88: Added sanitization of CHANGED_FILES before writing to $GITHUB_OUTPUT. The value is now piped through `printf '%s' "$CHANGED_FILES" | tr -d '\n\r'` to strip actual newline and carriage-return bytes, preventing injection of additional key=value pairs into GITHUB_OUTPUT via a malicious INPUT_SEPARATOR containing real newline bytes.

