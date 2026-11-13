<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.65.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.65.0** was hardened automatically. 11 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

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

**Notes:**

I changed nothing. All 11 'prior-fix' findings point at files that are relative symlinks inside the repo, all under src/usr/bin/ and src/usr/share/opentelemetry_shell/. The previous pass's 'fix' was to replace each symlink with a plain copy of the file it points to.

That copying doesn't make anything safer. For otel4netcat, otel4netcat_handler and otel4netcat_http, the copies in the old patches match the files the symlinks point to today, byte for byte. The other patches are cut off in the request, so I could only compare the parts shown, and those match upstream too. None of the patches add a security change.

All the symlinks resolve inside the action's own directory, so there is no security issue to fix. Keeping them leaves upstream's file layout and behaviour as they were.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

deploy/action.yml: the commit message in the Push step and the PR number in the 'Enable auto-merge' step now come in through env (COMMIT_MESSAGE, PR_NUMBER) and are double-quoted.

deploy-local/action.yml:
- Every input, github.* value (workflow, repository, repository_owner) and step output used in a run block now comes in through a per-step env: entry: WORKFLOWS_DIRECTORY, SELF_PATH, DEPLOY_REPOSITORY, DEPLOY_VERSION, INPUT_GITHUB_TOKEN, plus the rest.
- Where a value sat inside a single-quoted yq filter or echo template, I spliced it in as '"$VAR"' so the filter and template text stay exactly as before.
- Find self and the three Find-*-level steps strip newlines from each path with printf '%s' | tr -d '\n\r' before writing it.
- In Determine repository and Determine version, a while-read loop replaces `xargs -I '{}' echo key='{}'`. It strips newlines from each line before writing, so a value that spans several lines still produces one output line per line, as upstream's did.
- Determine version comment strips newlines from the comment before writing it.
- Upstream refers to an input that doesn't exist, inputs.check_suite_instrumentation_file_name, which evaluates to an empty string. I moved it into env unchanged so the behaviour stays the same.

