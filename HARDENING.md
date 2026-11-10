<!-- markdownlint-disable -->

# Hardening Report: tj-actions--verify-changed-files/v20.0.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--verify-changed-files/v20.0.3** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

entrypoint.sh, which the composite step 'verify-changed-files' runs, writes `echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"` with no `printf '%s' ... | tr -d '\n\r'` step first. CHANGED_FILES is built by joining file names with INPUT_SEPARATOR, which comes from `inputs.separator`. The script only escapes the literal two-character sequences `\n` and `\r` in the separator. Real newline or carriage-return characters pass through, so a caller-supplied separator can end the line and add extra output entries. Fix: strip newlines first, e.g. `safe=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')`, or write the value between random heredoc delimiters.

Locations:

- `entrypoint.sh:95`
- `action.yml:75`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 9386251..5aee8ca 100644
--- a/action.yml
+++ b/action.yml
@@ -45,7 +45,6 @@ inputs:
     description: "Use non-ASCII characters to match files and output the filenames completely verbatim by setting this to `false`"
     default: "true"
     required: false
-
 outputs:
   files_changed:
     description: Boolean indicating that files have changed.
@@ -53,12 +52,28 @@ outputs:
   changed_files:
     description: List of changed files
     value: ${{ steps.verify-changed-files.outputs.changed_files }}
-
 runs:
   using: 'composite'
   steps:
     - name: Glob match
       uses: tj-actions/glob@2deae40528141fc53131606d56b4e4ce2a486b29 # v22.0.2
       if: inputs.files != ''
       id: glob
       with:
@@ -69,7 +84,6 @@ runs:
         read-gitignore: ${{ inputs.read-gitignore }}
         match-gitignore-files: ${{ inputs.match-gitignore-files }}
         safe-output: ${{ inputs.safe_output }}
-      
     - run: |
         bash $GITHUB_ACTION_PATH/entrypoint.sh
       id: verify-changed-files
@@ -88,7 +102,6 @@ runs:
         INPUT_SAFE_OUTPUT: ${{ inputs.safe_output }}
         INPUT_PATH: ${{ inputs.path }}
         INPUT_QUOTEPATH: ${{ inputs.quotepath }}
-        
 branding:
   icon: file-text
   color: white

```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed entrypoint.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/entrypoint.sh b/entrypoint.sh
index fa1931e..8f00b65 100755
--- a/entrypoint.sh
+++ b/entrypoint.sh
@@ -92,7 +92,8 @@ if [[ -n "$CHANGED_FILES" ]]; then
   CHANGED_FILES=$(echo "$CHANGED_FILES" | awk '{gsub(/\|/,"\n"); print $0;}' | awk -v d="$INPUT_SEPARATOR" '{s=(NR==1?s:s d)$0}END{print s}')
 
   echo "files_changed=true" >> "$GITHUB_OUTPUT"
-  echo "changed_files=$CHANGED_FILES" >> "$GITHUB_OUTPUT"
+  safe_changed_files=$(printf '%s' "$CHANGED_FILES" | tr -d '\n\r')
+  echo "changed_files=$safe_changed_files" >> "$GITHUB_OUTPUT"
 
   if [[ "$INPUT_FAIL_IF_CHANGED" == "true" ]]; then
     if [[ -n "$INPUT_FAIL_MSG" ]]; then

```

Locations:

- `entrypoint.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection, prior-fix

**Notes:**

entrypoint.sh: CHANGED_FILES now has newline and carriage-return characters removed (printf '%s' | tr -d '\n\r') before it is written to GITHUB_OUTPUT. This blocks a real newline in the separator input from adding extra output entries. action.yml: re-applied the earlier harden's changes, which were only blank-line and whitespace cleanup. I also put quotes around $GITHUB_ACTION_PATH in the entrypoint call. tj-actions/glob was already pinned to a full SHA (v22.0.2), so nothing changed there. Nothing was run or tested.

