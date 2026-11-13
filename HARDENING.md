<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.66.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.66.0** was hardened automatically. 30 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): in the 'Push' step, `git commit -m "${{ inputs.commit_message }}"` puts the input straight into the bash script. A commit message containing `"$(...)"` runs commands, with GH_TOKEN, a token that can push, in the environment. Move it to env (COMMIT_MESSAGE) and use "$COMMIT_MESSAGE".

Locations:

- `actions/instrument/deploy/action.yml:195`

### script-injection (severity: high)

Rule (a): in the 'Enable auto-merge' step, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` puts a step output directly into the run: script, and it is also unquoted.

Locations:

- `actions/instrument/deploy/action.yml:220`

### script-injection (severity: high)

Rule (a): the 'Find self' step puts ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }} and ${{ inputs.workflows_directory }} directly into the bash script, e.g. `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `elif [ -r "${{ github.workflow }}" ]`.

Locations:

- `actions/instrument/deploy-local/action.yml:96`

### script-injection (severity: high)

Rule (a): the 'Determine repository' step puts ${{ steps.find-self.outputs.path }} and ${{ inputs.action_repository }} directly into the script, e.g. `if [ -n "${{ inputs.action_repository }}" ]; then echo '${{ inputs.action_repository }}'`.

Locations:

- `actions/instrument/deploy-local/action.yml:111`

### script-injection (severity: high)

Rule (a): the 'Determine version' step puts ${{ inputs.action_version }}, ${{ steps.find-self.outputs.path }} and ${{ steps.determine-repository.outputs.repository }} directly into the script, e.g. `echo '${{ inputs.action_version }}'`.

Locations:

- `actions/instrument/deploy-local/action.yml:129`

### script-injection (severity: high)

Rule (a): the 'Determine version comment' step puts ${{ inputs.action_version }} and ${{ steps.find-self.outputs.path }} directly into the script, e.g. `comment='${{ inputs.action_version }}'`.

Locations:

- `actions/instrument/deploy-local/action.yml:138`

### script-injection (severity: high)

Rule (a): the 'Canonicalize' step puts ${{ inputs.workflows_directory }} and ${{ steps.determine-repository.outputs.repository }} directly into the script, including inside a sed expression: `sed -i 's~'"$repository"'~${{ steps.determine-repository.outputs.repository }}~g'`.

Locations:

- `actions/instrument/deploy-local/action.yml:148`

### script-injection (severity: high)

Rule (a): the steps 'Find workflow-level observability', 'Find check-suite-level observability' and 'Find repository-level observability' all put ${{ inputs.workflows_directory }} directly into the script: `(ls "${{ inputs.workflows_directory }}"/*.yaml ...)`.

Locations:

- `actions/instrument/deploy-local/action.yml:157`
- `actions/instrument/deploy-local/action.yml:164`
- `actions/instrument/deploy-local/action.yml:171`

### script-injection (severity: high)

Rule (a): the steps 'Deploy workflow-level observability', 'Deploy check suite-level instrumentation' and 'Deploy repository-level instrumentation' put ${{ inputs.workflow_level_instrumentation_workflow_name }}, ${{ inputs.workflows_directory }}, ${{ inputs.*_file_name }}, ${{ steps.find-self.outputs.path }}, ${{ steps.determine-repository.outputs.repository }} and ${{ steps.determine-instrumentation-version.outputs.version }} directly into the scripts, e.g. `echo 'name: ${{ inputs.workflow_level_instrumentation_workflow_name }}`.

Locations:

- `actions/instrument/deploy-local/action.yml:179`
- `actions/instrument/deploy-local/action.yml:250`
- `actions/instrument/deploy-local/action.yml:285`

### script-injection (severity: high)

Rule (a): the three '... startup optimization' steps (workflow, check suite, repository) put ${{ steps.*.outputs.path }}, ${{ inputs.workflows_directory }}, ${{ github.repository_owner }} and ${{ inputs.github_token }} directly into the scripts. `version=${{ steps.determine-instrumentation-version.outputs.version }}` is unquoted, and `curl ... --header "Authorization: Bearer ${{ inputs.github_token }}"` writes the token itself into the script text.

Locations:

- `actions/instrument/deploy-local/action.yml:199`
- `actions/instrument/deploy-local/action.yml:268`
- `actions/instrument/deploy-local/action.yml:305`

### script-injection (severity: high)

Rule (a): the 'Update workflow-level observability triggers' step puts ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.workflows_directory }}, ${{ steps.find-*.outputs.path }}, ${{ inputs.github_token }} and ${{ github.repository }} directly into the script, e.g. `grep -qvF "$(echo "${{ inputs.workflow_level_instrumentation_exclude }}" | ...)"`.

Locations:

- `actions/instrument/deploy-local/action.yml:215`

### script-injection (severity: high)

Rule (a): the steps 'Deploy Copilot Setup', 'Deploy job-level instrumentation', 'Configure job-level instrumentation secret redaction', 'Modify permissions for job-level instrumentations', 'Remove explicit trace context declarations for job-level instrumentation', 'Annotate instrumentation action versions' and 'Restore blank lines' put ${{ inputs.workflows_directory }}, ${{ inputs.job_level_instrumentation_exclude }}, ${{ inputs.job_level_instrumentation_secret_redaction_strategy }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }} and ${{ steps.determine-instrumentation-version-comment.outputs.comment }} directly into the scripts, e.g. `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"` and `comment='${{ steps.determine-instrumentation-version-comment.outputs.comment }}'`.

Locations:

- `actions/instrument/deploy-local/action.yml:323`
- `actions/instrument/deploy-local/action.yml:340`
- `actions/instrument/deploy-local/action.yml:365`
- `actions/instrument/deploy-local/action.yml:395`
- `actions/instrument/deploy-local/action.yml:415`
- `actions/instrument/deploy-local/action.yml:430`
- `actions/instrument/deploy-local/action.yml:443`

### github-env-injection (severity: high)

The 'Find self' step writes `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. Neither value goes through `printf '%s' ... | tr -d '\n\r'` first, so a newline in either one adds extra output keys.

Locations:

- `actions/instrument/deploy-local/action.yml:98`

### github-env-injection (severity: high)

The 'Determine repository' step echoes `'${{ inputs.action_repository }}'` (or $GITHUB_ACTION_REPOSITORY, or a value read from the workflow file) and pipes it into `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"`. Newlines are never stripped.

Locations:

- `actions/instrument/deploy-local/action.yml:125`

### github-env-injection (severity: high)

The 'Determine version' step writes `${{ inputs.action_version }}`, or a value parsed from the workflow file, through `xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"`. Newlines are never stripped.

Locations:

- `actions/instrument/deploy-local/action.yml:134`

### github-env-injection (severity: high)

The 'Determine version comment' step builds `comment` from `${{ inputs.action_version }}` or from grep/sed on the workflow file, then writes `echo "comment=# $comment" >> "$GITHUB_OUTPUT"`. Newlines are never stripped.

Locations:

- `actions/instrument/deploy-local/action.yml:145`

### github-env-injection (severity: high)

The steps 'Find workflow-level observability', 'Find check-suite-level observability' and 'Find repository-level observability' write file names listed from `${{ inputs.workflows_directory }}` with `echo path="$workflow_file" >> "$GITHUB_OUTPUT"`. Newlines are never stripped.

Locations:

- `actions/instrument/deploy-local/action.yml:159`
- `actions/instrument/deploy-local/action.yml:166`
- `actions/instrument/deploy-local/action.yml:173`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/instrument/deploy/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/instrument/deploy/action.yml b/actions/instrument/deploy/action.yml
index 2ca6e46..cecb226 100644
--- a/actions/instrument/deploy/action.yml
+++ b/actions/instrument/deploy/action.yml
@@ -192,10 +192,11 @@ runs:
         git config user.email "${user_id}+${user_name}@users.noreply.github.com"
         git remote set-url origin "https://x-access-token:${GH_TOKEN}@github.com/${GITHUB_REPOSITORY}.git"
         git add -A
-        git commit -m "${{ inputs.commit_message }}"
+        git commit -m "$COMMIT_MESSAGE"
         git push && echo "pushed=true" >> "$GITHUB_OUTPUT" || echo "pushed=false" >> "$GITHUB_OUTPUT"
       env:
         GH_TOKEN: ${{ inputs.github_token }}
+        COMMIT_MESSAGE: ${{ inputs.commit_message }}
     - name: "Open Pull Request"
       if: ${{ inputs.dry_run != 'true' && steps.push.outputs.pushed == 'false' }}
       id: open-pr
@@ -217,7 +218,8 @@ runs:
     - name: "Enable auto-merge"
       if: steps.open-pr.outputs.pull-request-number != null
       shell: bash
-      run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}
+      run: gh pr merge --squash --auto "$PR_NUMBER"
       env:
         GH_TOKEN: ${{ inputs.github_token }}
+        PR_NUMBER: ${{ steps.open-pr.outputs.pull-request-number }}
       continue-on-error: true

```

Locations:

- `actions/instrument/deploy/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed actions/instrument/deploy-local/action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/actions/instrument/deploy-local/action.yml b/actions/instrument/deploy-local/action.yml
index f7c6f1e..3dc5d56 100644
--- a/actions/instrument/deploy-local/action.yml
+++ b/actions/instrument/deploy-local/action.yml
@@ -93,26 +93,34 @@ runs:
     - name: "Find self"
       id: find-self
       shell: bash
+      env:
+        PATH_OVERRIDE: ${{ inputs.__repository_level_instrumentation_file_name_override }}
+        WORKFLOW_NAME: ${{ github.workflow }}
+        WORKFLOWS_DIR: ${{ inputs.workflows_directory }}
       run: |
-        if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]; then
-          echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"
-        elif [ -r "${{ github.workflow }}" ]; then
-          echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"
+        if [ -r "$PATH_OVERRIDE" ]; then
+          safe="$(printf '%s' "$PATH_OVERRIDE" | tr -d '\n\r')"
+          echo path="$safe" >> "$GITHUB_OUTPUT"
+        elif [ -r "$WORKFLOW_NAME" ]; then
+          safe="$(printf '%s' "$WORKFLOW_NAME" | tr -d '\n\r')"
+          echo path="$safe" >> "$GITHUB_OUTPUT"
         else
-          (ls "${{ inputs.workflows_directory }}"/*.yaml "${{ inputs.workflows_directory }}"/*.yml 2> /dev/null || true) | while read -r workflow_file; do
-            [ "$(cat "$workflow_file" | yq .name)" = "${{ github.workflow }}" ] && echo path="$workflow_file" >> "$GITHUB_OUTPUT" && break || true
+          (ls "$WORKFLOWS_DIR"/*.yaml "$WORKFLOWS_DIR"/*.yml 2> /dev/null || true) | while read -r workflow_file; do
+            [ "$(cat "$workflow_file" | yq .name)" = "$WORKFLOW_NAME" ] && safe="$(printf '%s' "$workflow_file" | tr -d '\n\r')" && echo path="$safe" >> "$GITHUB_OUTPUT" && break || true
           done
         fi
     - name: "Determine repository"
       id: determine-repository
       env:
         GITHUB_ACTION_REPOSITORY: ${{ github.action_repository }}
+        SELF_PATH: ${{ steps.find-self.outputs.path }}
+        INPUT_ACTION_REPOSITORY: ${{ inputs.action_repository }}
       shell: bash
       run: |
-        uses="$(cat "${{ steps.find-self.outputs.path }}" | yq '.jobs[].steps[] | select(.uses == "*/actions/instrument/deploy*") | .uses' -r | head -n 1)"
+        uses="$(cat "$SELF_PATH" | yq '.jobs[].steps[] | select(.uses == "*/actions/instrument/deploy*") | .uses' -r | head -n 1)"
         self="${uses%%/actions/instrument/deploy*}"
-        if [ -n "${{ inputs.action_repository }}" ]; then
-          echo '${{ inputs.action_repository }}'
+        raw="$(if [ -n "$INPUT_ACTION_REPOSITORY" ]; then
+          printf '%s\n' "$INPUT_ACTION_REPOSITORY"
         elif [ -n "${GITHUB_ACTION_REPOSITORY:-}" ] && [ "${GITHUB_ACTION_REPOSITORY%/*}" != plengauer ]; then
           echo "$GITHUB_ACTION_REPOSITORY"
         elif [ "${self%/*}" != plengauer ]; then
@@ -122,62 +130,90 @@ runs:
         else
           echo "::notice::On private GitHub servers, action runners are not following redirects." >&2
           echo plengauer/Thoth
-        fi | xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"
+        fi)"
+        repository="$(printf '%s\n' "$raw" | sed '/^[[:space:]]*$/d' | tail -n 1 | tr -d '\n\r')"
+        [ -z "$repository" ] || echo "repository=$repository" >> "$GITHUB_OUTPUT"
     - name: "Determine version"
       id: determine-instrumentation-version
       shell: bash
+      env:
+        INPUT_ACTION_VERSION: ${{ inputs.action_version }}
+        SELF_PATH: ${{ steps.find-self.outputs.path }}
+        ACTION_REPO: ${{ steps.determine-repository.outputs.repository }}
       run: |
-        if [ "${{ inputs.action_version }}" = same ]; then
-          cat "${{ steps.find-self.outputs.path }}" | yq -r '.jobs[].steps[] | select(.uses == "${{ steps.determine-repository.outputs.repository }}/actions/instrument/deploy*") | .uses' | cut -d @ -f 2
+        raw="$(if [ "$INPUT_ACTION_VERSION" = same ]; then
+          cat "$SELF_PATH" | yq -r '.jobs[].steps[] | select(.uses == "'"$ACTION_REPO"'/actions/instrument/deplo
... (truncated)
```

Locations:

- `actions/instrument/deploy-local/action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/opentelemetry_shell.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/opentelemetry_shell.sh b/src/usr/bin/opentelemetry_shell.sh
deleted file mode 120000
index ab9809c..0000000
--- a/src/usr/bin/opentelemetry_shell.sh
+++ /dev/null
@@ -1 +0,0 @@
-otel.sh
\ No newline at end of file
diff --git a/src/usr/bin/opentelemetry_shell.sh b/src/usr/bin/opentelemetry_shell.sh
new file mode 100755
index 0000000..14b95b6
--- /dev/null
+++ b/src/usr/bin/opentelemetry_shell.sh
@@ -0,0 +1,566 @@
+#!/bin/false
+# shellcheck shell=bash
+################################################################################################################
+# This file is doing auto-instrumentation, auto-injection and auto-context-propagation.                        #
+# It should be sourced at the very top of any shell script that should be observed.                            #
+# Only use the "otel_instrument" and "otel_outstrument" functions directly.                                    #
+# All other functions and variables are for internal use only and therefore subject to change without notice!  #
+################################################################################################################
+
+if \[ "${_otel_shell_injected:-FALSE}" = "TRUE" ]; then
+  return 0
+fi
+_otel_shell_injected=TRUE
+
+case "$-" in
+  *r*)
+    \echo "WARNING OpenTelemetry for shell does not support restricted mode (set -r)!" >&2
+    return 0
+    ;;
+esac
+
+_otel_shell_conservative_exec="${OTEL_SHELL_CONSERVATIVE_EXEC:-FALSE}"
+unset OTEL_SHELL_CONSERVATIVE_EXEC
+
+\. /usr/share/opentelemetry_shell/api.sh
+_otel_package_version opentelemetry-shell >/dev/null # to build the cache outside a subshell
+
+if \[ "$_otel_shell" = "bash" ] && \[ -n "${BASHPID:-}" ] && \[ "$$" != "$BASHPID" ]; then
+  \echo "WARNING The OpenTelemetry shell file for auto-instrumentation is sourced in a subshell, automatic instrumentation will only be active within that subshell!" >&2
+fi
+
+case "$-" in
+  *i*) _otel_is_interactive=TRUE ;;
+  *) _otel_is_interactive=FALSE ;;
+esac
+
+if \[ -n "${OTEL_SHELL_AUTO_INSTRUMENTATION_HINT:-}" ]; then
+  _otel_shell_auto_instrumentation_hint="$OTEL_SHELL_AUTO_INSTRUMENTATION_HINT"
+  unset OTEL_SHELL_AUTO_INSTRUMENTATION_HINT
+elif \[ "$_otel_is_interactive" = TRUE ]; then
+  \echo "WARNING When using OpenTelemetry in an interactive shell for the first time after startup, it may take some time to create the instrumentation cache! Subsequent interactive shells will start faster. This performance impact does NOT apply in non-interactive shells, like scripts or invocations with -c." >&2
+  _otel_shell_auto_instrumentation_hint=""
+elif \[ "${OTEL_SHELL_IS_DYNAMIC:-}" = TRUE ]; then
+  _otel_shell_auto_instrumentation_hint=""
+  unset OTEL_SHELL_IS_DYNAMIC
+elif \[ -f "$0" ] && \[ "$(\readlink -f "$0")" != "$(\readlink -f "/proc/$$/exe")" ]; then
+  _otel_shell_auto_instrumentation_hint="$0"
+else
+  _otel_shell_auto_instrumentation_hint="$(_otel_resolve_command_self)"
+fi
+
+if \[ "$_otel_shell" = "bash" ]; then
+  _otel_source_file_resolver='${BASH_SOURCE:-}'
+else
+  _otel_source_file_resolver='$0'
+fi
+_otel_source_line_resolver='${LINENO:-}'
+_otel_source_func_resolver='${FUNCNAME:-}'
+
+if \[ "$_otel_shell" = "bash" ]; then
+  shopt -s expand_aliases 1>/dev/null 2>/dev/null
+fi
+
+_otel_auto_instrument() {
+  local hint="$1"
+  local IFS=' 
+'
+
+  # cached?
+  ## we really have three options for the cache key
+  ## (1) using the hint - will not work when scripts are changing or called the same but very fast!
+  ## (2) using the resolved hint - will not work when new executables are added onto the system or their shebang changes or new bash.rc aliases are added
+  ## (3) using the filtered list of commands - will work in every case but slowest
+  local cache_key="$({
+    \echo "$_otel_shell_conservative_exec" "${OTEL_SHELL_CONFIG_MUTE_INTERNALS:-}" "${OTEL_SHELL_CONFIG_MUTE_BUILTINS:-}" "$PATH" "$hint"
+    if \type dpkg 1>/dev/null 2>/dev/null; then \ls -la /var/lib/dpkg/lock; else _otel_list_path_commands | _otel_filter_commands_by_special | _otel_filter_commands_by_h
... (truncated)
```

Locations:

- `src/usr/bin/opentelemetry_shell.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/opentelemetry_shell_api.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/opentelemetry_shell_api.sh b/src/usr/bin/opentelemetry_shell_api.sh
deleted file mode 120000
index 82d0909..0000000
--- a/src/usr/bin/opentelemetry_shell_api.sh
+++ /dev/null
@@ -1 +0,0 @@
-otelapi.sh
\ No newline at end of file
diff --git a/src/usr/bin/opentelemetry_shell_api.sh b/src/usr/bin/opentelemetry_shell_api.sh
new file mode 100644
index 0000000..38d77f2
--- /dev/null
+++ b/src/usr/bin/opentelemetry_shell_api.sh
@@ -0,0 +1,593 @@
+#!/bin/false
+# shellcheck shell=bash
+##################################################################################################
+# This file is providing an API for creating and managing open telemetry spans.                  #
+# It should be sourced at the very top of any shell script where the functions are to be used.   #
+# All variables are for internal use only and therefore subject to change without notice!        #
+##################################################################################################
+
+# translate legacy config
+if \[ -n "${OTEL_SHELL_TRACES_ENABLE:-}" ] || \[ -n "${OTEL_SHELL_METRICS_ENABLE:-}" ] || \[ -n "${OTEL_SHELL_LOGS_ENABLE:-}" ]; then
+  \[ "${OTEL_SHELL_TRACES_ENABLE:-FALSE}" = TRUE ] && \export OTEL_TRACES_EXPORTER=otlp || \export OTEL_TRACES_EXPORTER=""
+  \[ "${OTEL_SHELL_METRICS_ENABLE:-FALSE}" = TRUE ] && \export OTEL_METRICS_EXPORTER=otlp || \export OTEL_METRICS_EXPORTER=""
+  \[ "${OTEL_SHELL_LOGS_ENABLE:-FALSE}" = TRUE ] && \export OTEL_LOGS_EXPORTER=otlp || \export OTEL_LOGS_EXPORTER=""
+  \export OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE="${OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE:-delta}"
+fi
+
+# check environment
+case "$-" in
+  *r*)
+    \echo "WARNING The OpenTelemetry shell API does not support restricted mode (set -r)!" >&2
+    exit 1
+    ;;
+esac
+
+# loadable builtins
+if \[ -n "${BASH_VERSION:-}" ] && \[ "${OTEL_SHELL_CONFIG_LOADABLE_BUILTINS:-TRUE}" = TRUE ]; then
+  \enable -f /usr/lib/bash/rm rm 2> /dev/null || \true
+  \enable -f /usr/lib/bash/mkfifo mkfifo 2> /dev/null || \true
+fi
+
+# basic setup
+if \[ -z "${TMPDIR:-}" ]; then TMPDIR=/tmp; fi
+_otel_shell_pipe_dir="${OTEL_SHELL_PIPE_DIR:-$TMPDIR}"
+_otel_remote_sdk_pipe="${OTEL_REMOTE_SDK_PIPE:-$(\mktemp -u -p "$_otel_shell_pipe_dir" opentelemetry_shell.$$.sdk.pipe.XXXXXXXXXX)}"
+_otel_remote_sdk_fd="${OTEL_REMOTE_SDK_FD:-7}"
+_otel_remote_sdk_stdout_redirect="${OTEL_SHELL_SDK_STDOUT_REDIRECT:-${OTEL_SHELL_SDK_OUTPUT_REDIRECT:-/dev/stderr}}"
+_otel_remote_sdk_stderr_redirect="${OTEL_SHELL_SDK_STDERR_REDIRECT:-${OTEL_SHELL_SDK_OUTPUT_REDIRECT:-/dev/stderr}}"
+if ! \[ -w "$_otel_remote_sdk_stdout_redirect" ]; then _otel_remote_sdk_stdout_redirect=/dev/null; fi
+if ! \[ -w "$_otel_remote_sdk_stderr_redirect" ]; then _otel_remote_sdk_stderr_redirect=/dev/null; fi
+_otel_shell="$(\readlink "/proc/$$/exe")"
+_otel_shell="${_otel_shell##*/}"
+if \[ "$_otel_shell" = busybox ]; then _otel_shell="busybox sh"; fi
+if \[ "${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}" = 0 ] || \[ "${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}" = "$PPID" ] || \[ "${PPID:-}" = 0 ] || { \[ -r /proc/$PPID/cmdline ] && \[ -r "/proc/${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}/cmdline" ] && \[ "$(\tr '\000-\037' ' ' </proc/$PPID/cmdline)" = "$(\tr '\000-\037' ' ' </proc/${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}/cmdline)" ]; }; then _otel_commandline_override="$OTEL_SHELL_COMMANDLINE_OVERRIDE"; fi
+unset OTEL_SHELL_COMMANDLINE_OVERRIDE
+unset OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE
+unset OTEL_SHELL_COMMAND_TYPE_OVERRIDE
+unset OTEL_SHELL_SPAN_KIND_OVERRIDE
+
+if \[ -p "$_otel_remote_sdk_pipe" ]; then
+  otel_init() {
+    _otel_mkfifo_flags="-m 666"
+    \eval "\\exec ${_otel_remote_sdk_fd}> \"$_otel_remote_sdk_pipe\""
+  }
+
+  otel_shutdown() {
+    \eval "\\exec ${_otel_remote_sdk_fd}>&-"
+  }
+else
+  otel_init() {
+    _otel_package_version opentelemetry-shell >/dev/null # to build the cache outside a subshell
+    _otel_package_version "$_otel_shell" >/dev/null
+    \mkfifo "$_otel_remote_sdk_pipe"
+    if \[ -n "${USER:-}
... (truncated)
```

Locations:

- `src/usr/bin/opentelemetry_shell_api.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/otel.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/otel.sh b/src/usr/bin/otel.sh
deleted file mode 120000
index e291cb5..0000000
--- a/src/usr/bin/otel.sh
+++ /dev/null
@@ -1 +0,0 @@
-../share/opentelemetry_shell/agent.sh
\ No newline at end of file
diff --git a/src/usr/bin/otel.sh b/src/usr/bin/otel.sh
new file mode 100755
index 0000000..14b95b6
--- /dev/null
+++ b/src/usr/bin/otel.sh
@@ -0,0 +1,566 @@
+#!/bin/false
+# shellcheck shell=bash
+################################################################################################################
+# This file is doing auto-instrumentation, auto-injection and auto-context-propagation.                        #
+# It should be sourced at the very top of any shell script that should be observed.                            #
+# Only use the "otel_instrument" and "otel_outstrument" functions directly.                                    #
+# All other functions and variables are for internal use only and therefore subject to change without notice!  #
+################################################################################################################
+
+if \[ "${_otel_shell_injected:-FALSE}" = "TRUE" ]; then
+  return 0
+fi
+_otel_shell_injected=TRUE
+
+case "$-" in
+  *r*)
+    \echo "WARNING OpenTelemetry for shell does not support restricted mode (set -r)!" >&2
+    return 0
+    ;;
+esac
+
+_otel_shell_conservative_exec="${OTEL_SHELL_CONSERVATIVE_EXEC:-FALSE}"
+unset OTEL_SHELL_CONSERVATIVE_EXEC
+
+\. /usr/share/opentelemetry_shell/api.sh
+_otel_package_version opentelemetry-shell >/dev/null # to build the cache outside a subshell
+
+if \[ "$_otel_shell" = "bash" ] && \[ -n "${BASHPID:-}" ] && \[ "$$" != "$BASHPID" ]; then
+  \echo "WARNING The OpenTelemetry shell file for auto-instrumentation is sourced in a subshell, automatic instrumentation will only be active within that subshell!" >&2
+fi
+
+case "$-" in
+  *i*) _otel_is_interactive=TRUE ;;
+  *) _otel_is_interactive=FALSE ;;
+esac
+
+if \[ -n "${OTEL_SHELL_AUTO_INSTRUMENTATION_HINT:-}" ]; then
+  _otel_shell_auto_instrumentation_hint="$OTEL_SHELL_AUTO_INSTRUMENTATION_HINT"
+  unset OTEL_SHELL_AUTO_INSTRUMENTATION_HINT
+elif \[ "$_otel_is_interactive" = TRUE ]; then
+  \echo "WARNING When using OpenTelemetry in an interactive shell for the first time after startup, it may take some time to create the instrumentation cache! Subsequent interactive shells will start faster. This performance impact does NOT apply in non-interactive shells, like scripts or invocations with -c." >&2
+  _otel_shell_auto_instrumentation_hint=""
+elif \[ "${OTEL_SHELL_IS_DYNAMIC:-}" = TRUE ]; then
+  _otel_shell_auto_instrumentation_hint=""
+  unset OTEL_SHELL_IS_DYNAMIC
+elif \[ -f "$0" ] && \[ "$(\readlink -f "$0")" != "$(\readlink -f "/proc/$$/exe")" ]; then
+  _otel_shell_auto_instrumentation_hint="$0"
+else
+  _otel_shell_auto_instrumentation_hint="$(_otel_resolve_command_self)"
+fi
+
+if \[ "$_otel_shell" = "bash" ]; then
+  _otel_source_file_resolver='${BASH_SOURCE:-}'
+else
+  _otel_source_file_resolver='$0'
+fi
+_otel_source_line_resolver='${LINENO:-}'
+_otel_source_func_resolver='${FUNCNAME:-}'
+
+if \[ "$_otel_shell" = "bash" ]; then
+  shopt -s expand_aliases 1>/dev/null 2>/dev/null
+fi
+
+_otel_auto_instrument() {
+  local hint="$1"
+  local IFS=' 
+'
+
+  # cached?
+  ## we really have three options for the cache key
+  ## (1) using the hint - will not work when scripts are changing or called the same but very fast!
+  ## (2) using the resolved hint - will not work when new executables are added onto the system or their shebang changes or new bash.rc aliases are added
+  ## (3) using the filtered list of commands - will work in every case but slowest
+  local cache_key="$({
+    \echo "$_otel_shell_conservative_exec" "${OTEL_SHELL_CONFIG_MUTE_INTERNALS:-}" "${OTEL_SHELL_CONFIG_MUTE_BUILTINS:-}" "$PATH" "$hint"
+    if \type dpkg 1>/dev/null 2>/dev/null; then \ls -la /var/lib/dpkg/lock; else _otel_list_path_commands | _otel_filter_commands_by_special | _otel_filter_commands_by_hint "$hint"; fi
+    \alias
+  } | \md5sum | \cut -d ' ' -f 
... (truncated)
```

Locations:

- `src/usr/bin/otel.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/otel4netcat without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/otel4netcat b/src/usr/bin/otel4netcat
deleted file mode 120000
index 59c3567..0000000
--- a/src/usr/bin/otel4netcat
+++ /dev/null
@@ -1 +0,0 @@
-../share/opentelemetry_shell/agent.injection.netcat.sh
\ No newline at end of file
diff --git a/src/usr/bin/otel4netcat b/src/usr/bin/otel4netcat
new file mode 100755
index 0000000..4cab456
--- /dev/null
+++ b/src/usr/bin/otel4netcat
@@ -0,0 +1,8 @@
+#!/bin/sh -e
+if [ "${OTEL_SHELL_AUTO_INJECTED:-FALSE}" = TRUE ]; then
+  exec "$@"
+else
+  . otelapi.sh
+  \eval "$(\grep -vh '_otel_alias_prepend ' /usr/share/opentelemetry_shell/agent.instrumentation.netcat.sh)"
+  _otel_inject_netcat "$@"
+fi

```

Locations:

- `src/usr/bin/otel4netcat`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/otel4netcat_handler without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/otel4netcat_handler b/src/usr/bin/otel4netcat_handler
deleted file mode 120000
index 4ef4b03..0000000
--- a/src/usr/bin/otel4netcat_handler
+++ /dev/null
@@ -1 +0,0 @@
-../share/opentelemetry_shell/agent.injection.netcat.handler.sh
\ No newline at end of file
diff --git a/src/usr/bin/otel4netcat_handler b/src/usr/bin/otel4netcat_handler
new file mode 100755
index 0000000..a243be8
--- /dev/null
+++ b/src/usr/bin/otel4netcat_handler
@@ -0,0 +1,38 @@
+#!/bin/sh -e
+. otelapi.sh
+handler_command_string="_otel_inject $(_otel_escape_args "$@")"
+netcat_command_string="$OTEL_SHELL_NETCAT_COMMAND"
+unset OTEL_SHELL_NETCAT_COMMAND
+export OTEL_SHELL_SDK_OUTPUT_REDIRECT="${OTEL_SHELL_SDK_OUTPUT_REDIRECT:-/dev/null}"
+OTEL_SHELL_AUTO_INSTRUMENTATION_HINT="$handler_command_string"
+OTEL_SHELL_AUTO_INJECTED=TRUE
+. otel.sh
+
+span_handle="$(otel_span_start CONSUMER send/receive)"
+_otel_netcat_parse_args 1 "$span_handle" $netcat_command_string >/dev/null
+otel_span_activate "$span_handle"
+
+if \[ "${OTEL_SHELL_CONFIG_NETCAT_ASSUME_REQUEST_RESPONSE:-FALSE}" = TRUE ]; then
+  exit_code_file="$(\mktemp)"
+  \echo 0 >"$exit_code_file"
+  span_handle_file_0="$(\mktemp -u)_opentelemetry_shell_$$.netcat.request.span_handle"
+  span_handle_file_1="$(\mktemp -u)_opentelemetry_shell_$$.netcat.command.span_handle"
+  span_handle_file_2="$(\mktemp -u)_opentelemetry_shell_$$.netcat.response.span_handle"
+  \mkfifo "$span_handle_file_0" "$span_handle_file_1" "$span_handle_file_2"
+  (\cat "$span_handle_file_0" | \tee "$span_handle_file_1" | \tee "$span_handle_file_2" >/dev/null &)
+  export OTEL_SHELL_CONFIG_NETCAT_READ_TIMEOUT=60
+  _otel_netcat_parse_request 1 "$span_handle_file_0" $netcat_command_string | {
+    unset OTEL_SHELL_CONFIG_NETCAT_READ_TIMEOUT
+    otel_span_activate "$(\cat "$span_handle_file_1")"
+    \eval "$handler_command_string" || \echo "$?" >"$exit_code_file"
+  } | _otel_netcat_parse_response 1 "$span_handle_file_2"
+  exit_code="$(\cat "$exit_code_file")"
+  \rm "$span_handle_file_0" "$span_handle_file_1" "$span_handle_file_2" "$exit_code_file" 2>/dev/null
+else
+  exit_code=0
+  \eval "$handler_command_string" || exit_code=$?
+fi
+
+otel_span_deactivate "$span_handle"
+otel_span_end "$span_handle"
+exit $exit_code

```

Locations:

- `src/usr/bin/otel4netcat_handler`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/otel4netcat_http without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/otel4netcat_http b/src/usr/bin/otel4netcat_http
deleted file mode 120000
index 917b372..0000000
--- a/src/usr/bin/otel4netcat_http
+++ /dev/null
@@ -1 +0,0 @@
-otel4netcat_request_response
\ No newline at end of file
diff --git a/src/usr/bin/otel4netcat_http b/src/usr/bin/otel4netcat_http
new file mode 100755
index 0000000..47f64ee
--- /dev/null
+++ b/src/usr/bin/otel4netcat_http
@@ -0,0 +1,3 @@
+#!/bin/sh -e
+export OTEL_SHELL_CONFIG_NETCAT_ASSUME_REQUEST_RESPONSE=TRUE
+exec otel4netcat "$@"

```

Locations:

- `src/usr/bin/otel4netcat_http`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/otel4sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/otel4sh b/src/usr/bin/otel4sh
deleted file mode 120000
index d0e7c97..0000000
--- a/src/usr/bin/otel4sh
+++ /dev/null
@@ -1 +0,0 @@
-../share/opentelemetry_shell/agent.injection.sh
\ No newline at end of file
diff --git a/src/usr/bin/otel4sh b/src/usr/bin/otel4sh
new file mode 100755
index 0000000..1e6dea4
--- /dev/null
+++ b/src/usr/bin/otel4sh
@@ -0,0 +1,6 @@
+#!/bin/sh
+export OTEL_SHELL_AUTO_INJECTED=TRUE
+. otel.sh
+unset OTEL_SHELL_AUTO_INJECTED
+\unalias env
+_otel_inject_env env "$@"

```

Locations:

- `src/usr/bin/otel4sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/bin/otelapi.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/bin/otelapi.sh b/src/usr/bin/otelapi.sh
deleted file mode 120000
index 0742ff8..0000000
--- a/src/usr/bin/otelapi.sh
+++ /dev/null
@@ -1 +0,0 @@
-../share/opentelemetry_shell/api.sh
\ No newline at end of file
diff --git a/src/usr/bin/otelapi.sh b/src/usr/bin/otelapi.sh
new file mode 100644
index 0000000..38d77f2
--- /dev/null
+++ b/src/usr/bin/otelapi.sh
@@ -0,0 +1,593 @@
+#!/bin/false
+# shellcheck shell=bash
+##################################################################################################
+# This file is providing an API for creating and managing open telemetry spans.                  #
+# It should be sourced at the very top of any shell script where the functions are to be used.   #
+# All variables are for internal use only and therefore subject to change without notice!        #
+##################################################################################################
+
+# translate legacy config
+if \[ -n "${OTEL_SHELL_TRACES_ENABLE:-}" ] || \[ -n "${OTEL_SHELL_METRICS_ENABLE:-}" ] || \[ -n "${OTEL_SHELL_LOGS_ENABLE:-}" ]; then
+  \[ "${OTEL_SHELL_TRACES_ENABLE:-FALSE}" = TRUE ] && \export OTEL_TRACES_EXPORTER=otlp || \export OTEL_TRACES_EXPORTER=""
+  \[ "${OTEL_SHELL_METRICS_ENABLE:-FALSE}" = TRUE ] && \export OTEL_METRICS_EXPORTER=otlp || \export OTEL_METRICS_EXPORTER=""
+  \[ "${OTEL_SHELL_LOGS_ENABLE:-FALSE}" = TRUE ] && \export OTEL_LOGS_EXPORTER=otlp || \export OTEL_LOGS_EXPORTER=""
+  \export OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE="${OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE:-delta}"
+fi
+
+# check environment
+case "$-" in
+  *r*)
+    \echo "WARNING The OpenTelemetry shell API does not support restricted mode (set -r)!" >&2
+    exit 1
+    ;;
+esac
+
+# loadable builtins
+if \[ -n "${BASH_VERSION:-}" ] && \[ "${OTEL_SHELL_CONFIG_LOADABLE_BUILTINS:-TRUE}" = TRUE ]; then
+  \enable -f /usr/lib/bash/rm rm 2> /dev/null || \true
+  \enable -f /usr/lib/bash/mkfifo mkfifo 2> /dev/null || \true
+fi
+
+# basic setup
+if \[ -z "${TMPDIR:-}" ]; then TMPDIR=/tmp; fi
+_otel_shell_pipe_dir="${OTEL_SHELL_PIPE_DIR:-$TMPDIR}"
+_otel_remote_sdk_pipe="${OTEL_REMOTE_SDK_PIPE:-$(\mktemp -u -p "$_otel_shell_pipe_dir" opentelemetry_shell.$$.sdk.pipe.XXXXXXXXXX)}"
+_otel_remote_sdk_fd="${OTEL_REMOTE_SDK_FD:-7}"
+_otel_remote_sdk_stdout_redirect="${OTEL_SHELL_SDK_STDOUT_REDIRECT:-${OTEL_SHELL_SDK_OUTPUT_REDIRECT:-/dev/stderr}}"
+_otel_remote_sdk_stderr_redirect="${OTEL_SHELL_SDK_STDERR_REDIRECT:-${OTEL_SHELL_SDK_OUTPUT_REDIRECT:-/dev/stderr}}"
+if ! \[ -w "$_otel_remote_sdk_stdout_redirect" ]; then _otel_remote_sdk_stdout_redirect=/dev/null; fi
+if ! \[ -w "$_otel_remote_sdk_stderr_redirect" ]; then _otel_remote_sdk_stderr_redirect=/dev/null; fi
+_otel_shell="$(\readlink "/proc/$$/exe")"
+_otel_shell="${_otel_shell##*/}"
+if \[ "$_otel_shell" = busybox ]; then _otel_shell="busybox sh"; fi
+if \[ "${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}" = 0 ] || \[ "${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}" = "$PPID" ] || \[ "${PPID:-}" = 0 ] || { \[ -r /proc/$PPID/cmdline ] && \[ -r "/proc/${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}/cmdline" ] && \[ "$(\tr '\000-\037' ' ' </proc/$PPID/cmdline)" = "$(\tr '\000-\037' ' ' </proc/${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}/cmdline)" ]; }; then _otel_commandline_override="$OTEL_SHELL_COMMANDLINE_OVERRIDE"; fi
+unset OTEL_SHELL_COMMANDLINE_OVERRIDE
+unset OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE
+unset OTEL_SHELL_COMMAND_TYPE_OVERRIDE
+unset OTEL_SHELL_SPAN_KIND_OVERRIDE
+
+if \[ -p "$_otel_remote_sdk_pipe" ]; then
+  otel_init() {
+    _otel_mkfifo_flags="-m 666"
+    \eval "\\exec ${_otel_remote_sdk_fd}> \"$_otel_remote_sdk_pipe\""
+  }
+
+  otel_shutdown() {
+    \eval "\\exec ${_otel_remote_sdk_fd}>&-"
+  }
+else
+  otel_init() {
+    _otel_package_version opentelemetry-shell >/dev/null # to build the cache outside a subshell
+    _otel_package_version "$_otel_shell" >/dev/null
+    \mkfifo "$_otel_remote_sdk_pipe"
+    if \[ -n "${USER:-}" ] && \[ -p /tmp/otel_shell/sdk_factory."$USER".pipe ] && \[ "${OTEL_L
... (truncated)
```

Locations:

- `src/usr/bin/otelapi.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/share/opentelemetry_shell/opentelemetry_shell.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/share/opentelemetry_shell/opentelemetry_shell.sh b/src/usr/share/opentelemetry_shell/opentelemetry_shell.sh
deleted file mode 120000
index a8ad122..0000000
--- a/src/usr/share/opentelemetry_shell/opentelemetry_shell.sh
+++ /dev/null
@@ -1 +0,0 @@
-agent.sh
\ No newline at end of file
diff --git a/src/usr/share/opentelemetry_shell/opentelemetry_shell.sh b/src/usr/share/opentelemetry_shell/opentelemetry_shell.sh
new file mode 100755
index 0000000..14b95b6
--- /dev/null
+++ b/src/usr/share/opentelemetry_shell/opentelemetry_shell.sh
@@ -0,0 +1,566 @@
+#!/bin/false
+# shellcheck shell=bash
+################################################################################################################
+# This file is doing auto-instrumentation, auto-injection and auto-context-propagation.                        #
+# It should be sourced at the very top of any shell script that should be observed.                            #
+# Only use the "otel_instrument" and "otel_outstrument" functions directly.                                    #
+# All other functions and variables are for internal use only and therefore subject to change without notice!  #
+################################################################################################################
+
+if \[ "${_otel_shell_injected:-FALSE}" = "TRUE" ]; then
+  return 0
+fi
+_otel_shell_injected=TRUE
+
+case "$-" in
+  *r*)
+    \echo "WARNING OpenTelemetry for shell does not support restricted mode (set -r)!" >&2
+    return 0
+    ;;
+esac
+
+_otel_shell_conservative_exec="${OTEL_SHELL_CONSERVATIVE_EXEC:-FALSE}"
+unset OTEL_SHELL_CONSERVATIVE_EXEC
+
+\. /usr/share/opentelemetry_shell/api.sh
+_otel_package_version opentelemetry-shell >/dev/null # to build the cache outside a subshell
+
+if \[ "$_otel_shell" = "bash" ] && \[ -n "${BASHPID:-}" ] && \[ "$$" != "$BASHPID" ]; then
+  \echo "WARNING The OpenTelemetry shell file for auto-instrumentation is sourced in a subshell, automatic instrumentation will only be active within that subshell!" >&2
+fi
+
+case "$-" in
+  *i*) _otel_is_interactive=TRUE ;;
+  *) _otel_is_interactive=FALSE ;;
+esac
+
+if \[ -n "${OTEL_SHELL_AUTO_INSTRUMENTATION_HINT:-}" ]; then
+  _otel_shell_auto_instrumentation_hint="$OTEL_SHELL_AUTO_INSTRUMENTATION_HINT"
+  unset OTEL_SHELL_AUTO_INSTRUMENTATION_HINT
+elif \[ "$_otel_is_interactive" = TRUE ]; then
+  \echo "WARNING When using OpenTelemetry in an interactive shell for the first time after startup, it may take some time to create the instrumentation cache! Subsequent interactive shells will start faster. This performance impact does NOT apply in non-interactive shells, like scripts or invocations with -c." >&2
+  _otel_shell_auto_instrumentation_hint=""
+elif \[ "${OTEL_SHELL_IS_DYNAMIC:-}" = TRUE ]; then
+  _otel_shell_auto_instrumentation_hint=""
+  unset OTEL_SHELL_IS_DYNAMIC
+elif \[ -f "$0" ] && \[ "$(\readlink -f "$0")" != "$(\readlink -f "/proc/$$/exe")" ]; then
+  _otel_shell_auto_instrumentation_hint="$0"
+else
+  _otel_shell_auto_instrumentation_hint="$(_otel_resolve_command_self)"
+fi
+
+if \[ "$_otel_shell" = "bash" ]; then
+  _otel_source_file_resolver='${BASH_SOURCE:-}'
+else
+  _otel_source_file_resolver='$0'
+fi
+_otel_source_line_resolver='${LINENO:-}'
+_otel_source_func_resolver='${FUNCNAME:-}'
+
+if \[ "$_otel_shell" = "bash" ]; then
+  shopt -s expand_aliases 1>/dev/null 2>/dev/null
+fi
+
+_otel_auto_instrument() {
+  local hint="$1"
+  local IFS=' 
+'
+
+  # cached?
+  ## we really have three options for the cache key
+  ## (1) using the hint - will not work when scripts are changing or called the same but very fast!
+  ## (2) using the resolved hint - will not work when new executables are added onto the system or their shebang changes or new bash.rc aliases are added
+  ## (3) using the filtered list of commands - will work in every case but slowest
+  local cache_key="$({
+    \echo "$_otel_shell_conservative_exec" "${OTEL_SHELL_CONFIG_MUTE_INTERNALS:-}" "${OTEL_SHELL_CONFIG_MUTE_BUILTINS:-}" "$PATH" "$hint"
+    if \type dpkg 1>/dev/null 2>/dev
... (truncated)
```

Locations:

- `src/usr/share/opentelemetry_shell/opentelemetry_shell.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh b/src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh
deleted file mode 120000
index df0ed18..0000000
--- a/src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh
+++ /dev/null
@@ -1 +0,0 @@
-api.sh
\ No newline at end of file
diff --git a/src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh b/src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh
new file mode 100644
index 0000000..38d77f2
--- /dev/null
+++ b/src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh
@@ -0,0 +1,593 @@
+#!/bin/false
+# shellcheck shell=bash
+##################################################################################################
+# This file is providing an API for creating and managing open telemetry spans.                  #
+# It should be sourced at the very top of any shell script where the functions are to be used.   #
+# All variables are for internal use only and therefore subject to change without notice!        #
+##################################################################################################
+
+# translate legacy config
+if \[ -n "${OTEL_SHELL_TRACES_ENABLE:-}" ] || \[ -n "${OTEL_SHELL_METRICS_ENABLE:-}" ] || \[ -n "${OTEL_SHELL_LOGS_ENABLE:-}" ]; then
+  \[ "${OTEL_SHELL_TRACES_ENABLE:-FALSE}" = TRUE ] && \export OTEL_TRACES_EXPORTER=otlp || \export OTEL_TRACES_EXPORTER=""
+  \[ "${OTEL_SHELL_METRICS_ENABLE:-FALSE}" = TRUE ] && \export OTEL_METRICS_EXPORTER=otlp || \export OTEL_METRICS_EXPORTER=""
+  \[ "${OTEL_SHELL_LOGS_ENABLE:-FALSE}" = TRUE ] && \export OTEL_LOGS_EXPORTER=otlp || \export OTEL_LOGS_EXPORTER=""
+  \export OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE="${OTEL_EXPORTER_OTLP_METRICS_TEMPORALITY_PREFERENCE:-delta}"
+fi
+
+# check environment
+case "$-" in
+  *r*)
+    \echo "WARNING The OpenTelemetry shell API does not support restricted mode (set -r)!" >&2
+    exit 1
+    ;;
+esac
+
+# loadable builtins
+if \[ -n "${BASH_VERSION:-}" ] && \[ "${OTEL_SHELL_CONFIG_LOADABLE_BUILTINS:-TRUE}" = TRUE ]; then
+  \enable -f /usr/lib/bash/rm rm 2> /dev/null || \true
+  \enable -f /usr/lib/bash/mkfifo mkfifo 2> /dev/null || \true
+fi
+
+# basic setup
+if \[ -z "${TMPDIR:-}" ]; then TMPDIR=/tmp; fi
+_otel_shell_pipe_dir="${OTEL_SHELL_PIPE_DIR:-$TMPDIR}"
+_otel_remote_sdk_pipe="${OTEL_REMOTE_SDK_PIPE:-$(\mktemp -u -p "$_otel_shell_pipe_dir" opentelemetry_shell.$$.sdk.pipe.XXXXXXXXXX)}"
+_otel_remote_sdk_fd="${OTEL_REMOTE_SDK_FD:-7}"
+_otel_remote_sdk_stdout_redirect="${OTEL_SHELL_SDK_STDOUT_REDIRECT:-${OTEL_SHELL_SDK_OUTPUT_REDIRECT:-/dev/stderr}}"
+_otel_remote_sdk_stderr_redirect="${OTEL_SHELL_SDK_STDERR_REDIRECT:-${OTEL_SHELL_SDK_OUTPUT_REDIRECT:-/dev/stderr}}"
+if ! \[ -w "$_otel_remote_sdk_stdout_redirect" ]; then _otel_remote_sdk_stdout_redirect=/dev/null; fi
+if ! \[ -w "$_otel_remote_sdk_stderr_redirect" ]; then _otel_remote_sdk_stderr_redirect=/dev/null; fi
+_otel_shell="$(\readlink "/proc/$$/exe")"
+_otel_shell="${_otel_shell##*/}"
+if \[ "$_otel_shell" = busybox ]; then _otel_shell="busybox sh"; fi
+if \[ "${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}" = 0 ] || \[ "${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}" = "$PPID" ] || \[ "${PPID:-}" = 0 ] || { \[ -r /proc/$PPID/cmdline ] && \[ -r "/proc/${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}/cmdline" ] && \[ "$(\tr '\000-\037' ' ' </proc/$PPID/cmdline)" = "$(\tr '\000-\037' ' ' </proc/${OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE:-}/cmdline)" ]; }; then _otel_commandline_override="$OTEL_SHELL_COMMANDLINE_OVERRIDE"; fi
+unset OTEL_SHELL_COMMANDLINE_OVERRIDE
+unset OTEL_SHELL_COMMANDLINE_OVERRIDE_SIGNATURE
+unset OTEL_SHELL_COMMAND_TYPE_OVERRIDE
+unset OTEL_SHELL_SPAN_KIND_OVERRIDE
+
+if \[ -p "$_otel_remote_sdk_pipe" ]; then
+  otel_init() {
+    _otel_mkfifo_flags="-m 666"
+    \eval "\\exec ${_otel_remote_sdk_fd}> \"$_otel_remote_sdk_pipe\""
+  }
+
+  otel_shutdown() {
+    \eval "\\exec ${_otel_remote_sdk_fd}>&-"
+  }
+else
+  otel_init() {
+    _otel_package_version opentelemetry-shell >/dev/null # to build the cache outsid
... (truncated)
```

Locations:

- `src/usr/share/opentelemetry_shell/opentelemetry_shell_api.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py b/src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py
deleted file mode 120000
index 800bb5a..0000000
--- a/src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py
+++ /dev/null
@@ -1 +0,0 @@
-sdk.py
\ No newline at end of file
diff --git a/src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py b/src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py
new file mode 100644
index 0000000..d36f031
--- /dev/null
+++ b/src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py
@@ -0,0 +1,614 @@
+import sys
+import os
+import time
+from datetime import datetime, timezone
+
+initialized_traces = False
+initialized_metrics = False
+initialized_logs = False
+
+resource = {}
+final_resources = None
+spans: dict = {}
+next_span_id = 0
+events = {}
+next_event_id = 0
+links = {}
+next_link_id = 0
+counters = {}
+next_counter_id = 0
+observations: dict = {}
+next_observation_id = 0
+delayed_observations = {}
+
+auto_end = False
+
+def main():
+    run(sys.argv[1], sys.argv[2], sys.stdin)
+
+def run(scope, version, commands):
+    for line in commands:
+        line = line.strip()
+        if len(line) == 0:
+            continue
+        tokens = line.split(' ', 1)
+        tokens = [token for token in tokens if token]
+        try:
+            handle(scope, version, tokens[0], tokens[1] if len(tokens) > 1 else None)
+        except EOFError:
+            sys.exit(0)
+        except Exception:
+            print('SDK Error: ' + line, file=sys.stderr)
+            import traceback
+            traceback.print_exc()
+    try:
+        handle(scope, version, 'SHUTDOWN', None)
+    except EOFError:
+        sys.exit(0)
+    except Exception:
+        pass
+
+def guess_cloud_resource_detectors():
+    if os.environ.get('GITHUB_ACTIONS', 'false') == 'true' and os.environ.get('RUNNER_ENVIRONMENT', 'unknown') == 'github-hosted':
+        from opentelemetry.resource.detector.azure.vm import AzureVMResourceDetector
+        return [ AzureVMResourceDetector() ]
+    elif file_contains('/sys/devices/virtual/dmi/id/product_uuid', 'ec2') or file_contains('/sys/hypervisor/uuid', 'ec2'):
+        from opentelemetry.sdk.extension.aws.resource.ec2 import AwsEc2ResourceDetector
+        return [ AwsEc2ResourceDetector() ]
+    elif os.path.exists('/var/lib/waagent/'):
+        from opentelemetry.resource.detector.azure.vm import AzureVMResourceDetector
+        return [ AzureVMResourceDetector() ]
+    elif os.path.exists('/etc/google_instance_config.json') or file_contains('/sys/class/dmi/id/product_name', 'Google Compute Engine'):
+        from opentelemetry.resourcedetector.gcp_resource_detector import GoogleCloudResourceDetector
+        return [ GoogleCloudResourceDetector() ]
+    elif file_contains('/sys/class/dmi/id/product_name', 'OracleCloud.com'):
+        from opentelemetry.sdk.resources import Resource, ResourceDetector
+        class OracleCloudResourceDetector(ResourceDetector):
+            def detect(self) -> Resource:
+                try:
+                    metadata = self.fetch_metadata()
+                    resource = Resource.create({
+                        "cloud.provider": "oracle",
+                        "cloud.region": metadata['region'],
+                        "cloud.availability_zone": metadata['availabilityDomain'],
+                        "cloud.account_id": metadata['tenantId'],
+                        "host.type": metadata['shape'],
+                        "host.name": metadata['hostname'],
+                        "host.id": metadata['id'],
+                        "host.image_id": metadata['image']
+                    })
+                    return resource
+                except Exception:
+                    return Resource({})
+            def fetch_metadata(self):
+                import requests
+                response = requests.get('http://169.254.169.254/opc/v1/instance/', headers={'Authorization': 'Bearer Oracle'}, timeout=0.1)
+                response.raise_for_status()  # Raise an exception
... (truncated)
```

Locations:

- `src/usr/share/opentelemetry_shell/opentelemetry_shell_sdk.py`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, prior-fix

**Notes:**

Nothing was run or tested; the only check was a search confirming no ${{ }} expression is left inside any run: script.

actions/instrument/deploy/action.yml: the commit message now reaches the 'Push' step as COMMIT_MESSAGE, and the PR number reaches 'Enable auto-merge' as PR_NUMBER, now quoted. Both go through env:.

actions/instrument/deploy-local/action.yml: in every step, each ${{ }} expression (inputs, step outputs, github.workflow, github.repository_owner, github.repository, inputs.github_token) moved into the step's env: block. The scripts now read these as shell variables.
- Where the action repository or version sits inside a single-quoted yq expression, a sed expression or a generated workflow file, I spliced the variable in with shell quoting ('"$ACTION_REPO"'). The resulting text is the same as before, but nothing in the value runs as shell code.
- The unquoted version= in the three startup-optimization steps is now quoted. The token now reaches curl from an env var instead of being written into the script.
- Two expressions point at an input this action does not have, inputs.check_suite_instrumentation_file_name, so they always come out empty. I kept them as they were through env: rather than change what the action does.

GITHUB_OUTPUT writes: path, repository, version and comment now go through printf '%s' | tr -d '\n\r' before they are written.
- 'Determine repository' and 'Determine version' used to pipe through xargs -I. They now capture the value, keep the last non-empty line and strip newlines. That matches the old result, where the last line written wins. Nothing is written when the value is empty, as before.

Prior-fix items for src/usr/bin/* and src/usr/share/opentelemetry_shell/*: the earlier patch only replaced these symlinks with copies of their targets inside the repo. That had no security effect, so I left them as symlinks.

