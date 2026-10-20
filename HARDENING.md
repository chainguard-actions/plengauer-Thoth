<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.64.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.64.0** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in actions/instrument/deploy-local/action.yml directly interpolate ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions inside shell commands (sub-rule a). Examples include: 'Find self' step uses ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} directly in shell conditionals and commands; 'Determine repository' step uses ${{ inputs.action_repository }} in an if-condition; 'Determine version' step uses ${{ inputs.action_version }} in a shell if-block; 'Determine version comment' step uses ${{ inputs.action_version }} in echo and grep commands; 'Canonicalize' step uses ${{ inputs.workflows_directory }} and ${{ steps.determine-repository.outputs.repository }} in sed commands; 'Find workflow-level observability', 'Find check-suite-level observability', and 'Find repository-level observability' steps use ${{ inputs.workflows_directory }} in ls commands; 'Deploy workflow-level observability' step uses ${{ inputs.workflow_level_instrumentation_workflow_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, and ${{ inputs.workflows_directory }} in shell heredocs and commands; 'Deploy job-level instrumentation' step uses ${{ inputs.job_level_instrumentation_exclude }}, ${{ steps.determine-repository.outputs.repository }}, and ${{ steps.determine-instrumentation-version.outputs.version }} in shell commands; 'Configure job-level instrumentation secret redaction' step uses ${{ inputs.job_level_instrumentation_secret_redaction_strategy }} and ${{ steps.determine-repository.outputs.repository }} in case statements; 'Annotate instrumentation action versions' step uses ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ steps.determine-instrumentation-version-comment.outputs.comment }}, ${{ inputs.workflows_directory }}, and ${{ steps.determine-repository.outputs.repository }} in shell commands. Any of these inputs can contain shell metacharacters that will be interpreted by the shell before quoting takes effect.

Locations:

- `actions/instrument/deploy-local/action.yml:83`
- `actions/instrument/deploy-local/action.yml:84`
- `actions/instrument/deploy-local/action.yml:85`
- `actions/instrument/deploy-local/action.yml:87`
- `actions/instrument/deploy-local/action.yml:93`
- `actions/instrument/deploy-local/action.yml:99`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:113`
- `actions/instrument/deploy-local/action.yml:119`
- `actions/instrument/deploy-local/action.yml:125`

### script-injection (severity: high)

Two run: blocks in actions/instrument/deploy/action.yml directly interpolate ${{ inputs.* }} and ${{ steps.*.outputs.* }} expressions inside shell commands (sub-rule a). (1) The 'Push' step uses ${{ inputs.commit_message }} directly in a git commit -m "..." shell command: 'git commit -m "${{ inputs.commit_message }}"'. A caller-controlled commit_message containing shell metacharacters or newlines could alter the command. (2) The 'Enable auto-merge' step uses ${{ steps.open-pr.outputs.pull-request-number }} directly in a gh pr merge command: 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}'. Step outputs are workflow-controllable and must not be interpolated directly into shell.

Locations:

- `actions/instrument/deploy/action.yml:163`
- `actions/instrument/deploy/action.yml:183`

### github-env-injection (severity: high)

Multiple run: blocks in actions/instrument/deploy-local/action.yml write values derived from untrusted inputs directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). (1) The 'Find self' step writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT: 'echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"' and 'echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"'. (2) The 'Find workflow-level observability', 'Find check-suite-level observability', and 'Find repository-level observability' steps write workflow_file paths (derived from ${{ inputs.workflows_directory }}) to $GITHUB_OUTPUT via 'echo path="$workflow_file" >> "$GITHUB_OUTPUT"' without sanitization. A newline injected via these inputs could add arbitrary key=value pairs to the GITHUB_OUTPUT file, enabling environment variable injection into subsequent steps.

Locations:

- `actions/instrument/deploy-local/action.yml:84`
- `actions/instrument/deploy-local/action.yml:86`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:113`
- `actions/instrument/deploy-local/action.yml:119`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in both action files:

1. actions/instrument/deploy/action.yml:
   - 'Push' step: moved inputs.commit_message to env var COMMIT_MESSAGE
   - 'Enable auto-merge' step: moved steps.open-pr.outputs.pull-request-number to env var PR_NUMBER

2. actions/instrument/deploy-local/action.yml:
   - 'Find self' step: moved all ${{ }} expressions to env vars; added tr -d '\n\r' sanitization before writing to $GITHUB_OUTPUT
   - 'Determine repository' step: moved inputs.action_repository and steps.find-self.outputs.path to env vars
   - 'Determine version' step: moved inputs.action_version, steps.find-self.outputs.path, steps.determine-repository.outputs.repository to env vars
   - 'Determine version comment' step: moved inputs.action_version and steps.find-self.outputs.path to env vars
   - 'Canonicalize' step: moved inputs.workflows_directory and steps.determine-repository.outputs.repository to env vars
   - 'Find workflow-level observability' step: moved inputs.workflows_directory to env var; added sanitization for $GITHUB_OUTPUT writes
   - 'Find check-suite-level observability' step: same as above
   - 'Find repository-level observability' step: same as above
   - 'Deploy workflow-level observability' step: moved all ${{ }} expressions to env vars
   - 'Deploy workflow-level startup optimization' step: moved all ${{ }} expressions to env vars
   - 'Update workflow-level observability triggers' step: moved all ${{ }} expressions to env vars
   - 'Deploy check suite-level instrumentation' step: moved all ${{ }} expressions to env vars
   - 'Deploy check suite-level startup optimization' step: moved all ${{ }} expressions to env vars
   - 'Deploy repository-level instrumentation' step: moved all ${{ }} expressions to env vars
   - 'Deploy repository-level startup optimization' step: moved all ${{ }} expressions to env vars
   - 'Deploy Copilot Setup' step: moved inputs.workflows_directory to env var
   - 'Deploy job-level instrumentation' step: moved all ${{ }} expressions to env vars
   - 'Configure job-level instrumentation secret redaction' step: moved all ${{ }} expressions to env vars
   - 'Modify permissions for job-level instrumentations' step: moved inputs.workflows_directory to env var
   - 'Remove explicit trace context declarations' step: moved inputs.workflows_directory to env var
   - 'Annotate instrumentation action versions' step: moved all ${{ }} expressions to env vars
   - 'Restore blank lines' step: moved inputs.workflows_directory to env var

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed all four github-env-injection findings:
1. deploy-local/action.yml 'Determine repository' step: replaced `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` with a read+sanitize pattern using `tr -d '\n\r'`.
2. deploy-local/action.yml 'Determine version' step: same fix for version output.
3. deploy-local/action.yml 'Determine version comment' step: added `safe_comment=$(printf '%s' "$comment" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
4. inject_and_init.sh: replaced direct `printenv | grep ... >> "$GITHUB_ENV"` with a while loop that sanitizes each variable's value with `tr -d '\n\r'` before writing to GITHUB_ENV.

### Iteration 3

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe pipe-to-shell pattern in hardened/action/actions/instrument/shared/install.sh at line 28. Changed `wget -O - https://raw.githubusercontent.com/plengauer/Thoth/main/INSTALL.sh | sh -e` to download the script to a temporary file first using `mktemp`, then execute it separately with `sh -e "$_install_sh"`, and clean up the temp file afterward. This prevents a compromised or man-in-the-middle server from executing arbitrary code directly through the pipe.

