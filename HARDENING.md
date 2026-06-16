<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tj-actions--verify-changed-files/v20.0.4** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh (invoked from the action.yml `run:` block), the variable `CHANGED_FILES` is written directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' | tr -d '\n\r'`). `CHANGED_FILES` is constructed by incorporating `$INPUT_SEPARATOR` — which is set from the untrusted `inputs.separator` input via the `env:` block in action.yml — into an awk command that joins filenames. While `INPUT_SEPARATOR` undergoes partial percent-encoding at the top of the script, this does not constitute the required sanitization. A malicious caller could supply a separator value containing newline characters (after bypassing the percent-encoding) to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting subsequent output values. The write at `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"` must be preceded by `safe=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')` and use `$safe` instead.

Locations:

- `entrypoint.sh:89`
- `entrypoint.sh:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed github-env-injection in entrypoint.sh at lines 89-90. Added sanitization step `safe=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, and replaced `$CHANGED_FILES` with `$safe` in the echo statement. This prevents newline injection attacks where a malicious caller could supply a separator value containing newline characters (after bypassing the percent-encoding) to inject arbitrary key=value pairs into $GITHUB_OUTPUT.

