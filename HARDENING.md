<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.3** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable CHANGED_FILES is written to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). CHANGED_FILES is constructed using INPUT_SEPARATOR (mapped from the caller-controlled input `inputs.separator` via the env: block in action.yml) as an awk field delimiter. Because INPUT_SEPARATOR is untrusted and can contain newline characters, an attacker can inject arbitrary key=value pairs into $GITHUB_OUTPUT by supplying a newline-containing separator. The offending line is: `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"`

Locations:

- `entrypoint.sh:97`
- `action.yml:64`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed entrypoint.sh line 97: Added sanitization of CHANGED_FILES before writing to $GITHUB_OUTPUT. The value is now passed through `printf '%s' "$CHANGED_FILES" | tr -d '\n\r'` to strip newline and carriage return characters before being written as `changed_files=...` to $GITHUB_OUTPUT. This prevents an attacker from injecting arbitrary key=value pairs into $GITHUB_OUTPUT by supplying a newline-containing separator via inputs.separator.

