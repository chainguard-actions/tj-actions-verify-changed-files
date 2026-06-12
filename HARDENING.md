<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **tj-actions--verify-changed-files/v20.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The composite action references `tj-actions/glob@v22` using a mutable tag (`v22`) instead of a pinned 40-character commit SHA. This means the dependency can be silently updated or compromised without any change to this action, enabling a supply-chain attack.

Locations:

- `action.yml:47`

### github-env-injection (severity: high)

In entrypoint.sh, the variable `CHANGED_FILES` is constructed using `INPUT_SEPARATOR` (sourced from `inputs.separator`, a caller-controlled value) via an awk command: `awk -v d="$INPUT_SEPARATOR" '{s=(NR==1?s:s d)$0}END{print s}'`. The result is then written directly to `$GITHUB_OUTPUT` as `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Although `INPUT_SEPARATOR` is percent-encoded for `%`, `.`, `\n`, and `\r` literals at the top of the script, an actual embedded newline character in the separator value could inject additional key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other outputs or injecting malicious values.

Locations:

- `entrypoint.sh:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, github-env-injection

**Notes:**

1. Pinned tj-actions/glob@v22 to full SHA 2deae40528141fc53131606d56b4e4ce2a486b29 in action.yml (line 47). 2. Fixed github-env-injection in entrypoint.sh by sanitizing CHANGED_FILES with `printf '%s' "$CHANGED_FILES" | tr -d '\n\r'` before writing to $GITHUB_OUTPUT, preventing newline injection attacks via the caller-controlled INPUT_SEPARATOR value.

