<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In entrypoint.sh, the variable CHANGED_FILES — assembled from git-tracked filenames joined with $INPUT_SEPARATOR (which is sourced from inputs.separator, a caller-controlled input) — is written directly to $GITHUB_OUTPUT without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A crafted filename or separator value containing embedded newlines could inject additional key=value pairs into $GITHUB_OUTPUT, potentially overwriting other step outputs. The offending line is: `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"`. No `tr -d` or `printf '%s'` sanitization exists anywhere in the script.

Locations:

- `entrypoint.sh:95`

### script-injection (severity: high)

Rule (b) violation: In entrypoint.sh, the env var $INPUT_PATH (sourced from inputs.path via the env: block in action.yml) is used unquoted in the bash conditional `if [[ -n $INPUT_PATH ]]; then`. Although bash's [[ ]] construct prevents word splitting, glob/pathname expansion still occurs on unquoted variables, allowing a caller-supplied path value containing glob metacharacters (e.g. `*`, `?`, `[`) to expand unexpectedly. The variable should be double-quoted: `if [[ -n "$INPUT_PATH" ]]; then`.

Locations:

- `entrypoint.sh:17`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two issues in hardened/action/entrypoint.sh:
1. script-injection (line 17): Quoted `$INPUT_PATH` in `[[ -n "$INPUT_PATH" ]]` to prevent glob/pathname expansion from a caller-supplied path value containing metacharacters.
2. github-env-injection (line 95): Added `safe_changed_files=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, so embedded newlines in filenames or the caller-controlled separator cannot inject additional key=value pairs into the output.

