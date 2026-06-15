<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.56.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.56.0** was hardened automatically. 15 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'Find self' step interpolates ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} directly into shell commands inside the run: block. Attacker-controlled input values containing shell metacharacters would be executed by the shell.

Locations:

- `actions/instrument/deploy/action.yml:106`

### script-injection (severity: high)

Rule (a): The 'Determine repository' step interpolates ${{ inputs.action_repository }} and ${{ steps.find-self.outputs.path }} directly into shell commands inside the run: block without quoting through the expression mechanism.

Locations:

- `actions/instrument/deploy/action.yml:117`

### script-injection (severity: high)

Rule (a): The 'Determine version' step interpolates ${{ inputs.action_version }} and ${{ steps.find-self.outputs.path }} directly into shell commands inside the run: block. The value is also written to $GITHUB_OUTPUT via xargs without sanitization.

Locations:

- `actions/instrument/deploy/action.yml:129`

### script-injection (severity: high)

Rule (a): The 'Canonicalize' step interpolates ${{ steps.determine-repository.outputs.repository }} and ${{ inputs.workflows_directory }} directly into a sed command: sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'. A malicious value could inject shell metacharacters.

Locations:

- `actions/instrument/deploy/action.yml:138`

### script-injection (severity: high)

Rule (a): The 'Find workflow-level observability', 'Find check-suite-level observability', and 'Find repository-level observability' steps interpolate ${{ inputs.workflows_directory }} directly into shell ls/cat commands and write results to $GITHUB_OUTPUT.

Locations:

- `actions/instrument/deploy/action.yml:143`
- `actions/instrument/deploy/action.yml:149`
- `actions/instrument/deploy/action.yml:155`

### script-injection (severity: high)

Rule (a): The 'Deploy workflow-level observability' step interpolates ${{ inputs.workflow_level_instrumentation_workflow_name }}, ${{ inputs.workflows_directory }}, ${{ inputs.workflow_level_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, and ${{ steps.determine-instrumentation-version.outputs.version }} directly into shell commands.

Locations:

- `actions/instrument/deploy/action.yml:162`

### script-injection (severity: high)

Rule (a): The 'Deploy workflow-level startup optimization' step interpolates ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} directly into shell commands including a curl Authorization header.

Locations:

- `actions/instrument/deploy/action.yml:178`

### script-injection (severity: high)

Rule (a): The 'Update workflow-level observability triggers' step interpolates ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.workflows_directory }}, ${{ inputs.github_token }}, ${{ github.repository }}, and multiple steps outputs directly into shell commands including grep and curl invocations.

Locations:

- `actions/instrument/deploy/action.yml:196`

### script-injection (severity: high)

Rule (a): The 'Deploy check suite-level instrumentation', 'Deploy check suite-level startup optimization', 'Deploy repository-level instrumentation', and 'Deploy repository-level startup optimization' steps all interpolate ${{ inputs.workflows_directory }}, ${{ inputs.check_suite_instrumentation_file_name }}, ${{ inputs.repository_level_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} directly into shell commands.

Locations:

- `actions/instrument/deploy/action.yml:228`
- `actions/instrument/deploy/action.yml:248`
- `actions/instrument/deploy/action.yml:265`
- `actions/instrument/deploy/action.yml:285`

### script-injection (severity: high)

Rule (a): The 'Deploy Copilot Setup' and 'Deploy job-level instrumentation' steps interpolate ${{ inputs.workflows_directory }}, ${{ inputs.job_level_instrumentation_exclude }}, ${{ steps.determine-repository.outputs.repository }}, and ${{ steps.determine-instrumentation-version.outputs.version }} directly into shell commands.

Locations:

- `actions/instrument/deploy/action.yml:302`
- `actions/instrument/deploy/action.yml:315`

### script-injection (severity: high)

Rule (a): The 'Configure job-level instrumentation secret redaction' step interpolates ${{ inputs.job_level_instrumentation_secret_redaction_strategy }} and ${{ steps.determine-repository.outputs.repository }} directly into a case statement and shell commands. An attacker-controlled strategy value could break out of the case statement.

Locations:

- `actions/instrument/deploy/action.yml:340`

### script-injection (severity: high)

Rule (a): The 'Push' step interpolates ${{ inputs.commit_message }} directly into a `git commit -m "${{ inputs.commit_message }}"` shell command. An attacker-controlled commit message containing shell metacharacters could inject arbitrary commands.

Locations:

- `actions/instrument/deploy/action.yml:420`

### script-injection (severity: high)

Rule (a): The 'Enable auto-merge' step interpolates ${{ steps.open-pr.outputs.pull-request-number }} directly into a `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` shell command. A malicious pull-request-number value could inject shell arguments or commands.

Locations:

- `actions/instrument/deploy/action.yml:450`

### github-env-injection (severity: high)

The 'Find self' step writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker-controlled value containing newlines could inject additional key=value pairs into the GitHub output context.

Locations:

- `actions/instrument/deploy/action.yml:107`
- `actions/instrument/deploy/action.yml:109`

### github-env-injection (severity: high)

The 'Determine version' step writes ${{ inputs.action_version }} to $GITHUB_OUTPUT via `xargs -I '{}' echo version='{}'` without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker-controlled action_version value containing newlines could inject additional key=value pairs into the GitHub output context.

Locations:

- `actions/instrument/deploy/action.yml:133`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 13 script-injection and 2 github-env-injection findings in actions/instrument/deploy/action.yml. All ${{ }} expressions that were directly interpolated into run: shell blocks have been moved to env: blocks and referenced as plain environment variables. Key changes: (1) 'Find self' step: moved __repository_level_instrumentation_file_name_override, github.workflow, and workflows_directory to env: block; added printf/tr sanitization before writing to $GITHUB_OUTPUT. (2) 'Determine repository' step: moved action_repository and find-self path to env: block. (3) 'Determine version' step: moved action_version and find-self path to env: block; used printf/tr sanitization for the version value. (4) 'Canonicalize' step: moved determine-repository and workflows_directory to env: block; replaced sed with env var reference. (5) 'Find workflow/check-suite/repository-level observability' steps: moved workflows_directory to env: blocks. (6) All 'Deploy' steps: moved all inputs and step outputs to env: blocks. (7) 'Configure job-level instrumentation secret redaction' step: moved strategy and repository to env: block; removed the ${{ inputs.job_level_instrumentation_secret_redaction_strategy }} from the error message. (8) 'Push' step: moved commit_message to env: block as COMMIT_MESSAGE. (9) 'Enable auto-merge' step: moved pull-request-number to env: block as PR_NUMBER and quoted it properly.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine repository' step in actions/instrument/deploy/action.yml. The vulnerable pattern `fi | xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` was replaced with `fi | { read -r _repo; safe=$(printf '%s' "$_repo" | tr -d '\n\r'); echo "repository=$safe" >> "$GITHUB_OUTPUT"; }`. This sanitizes the repository value by stripping newline and carriage-return characters before writing to GITHUB_OUTPUT, preventing an attacker from injecting arbitrary key=value pairs via the action_repository input or github.action_repository context value.

