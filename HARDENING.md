<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.58.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.58.0** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Find self' step directly interpolates ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} inside run: shell commands. These expressions are expanded by the template engine before the shell sees them, allowing an attacker-controlled value to inject arbitrary shell commands. Example: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`.

Locations:

- `actions/instrument/deploy/action.yml:102`

### script-injection (severity: high)

Sub-rule (a): The 'Determine repository' step directly interpolates ${{ inputs.action_repository }}, ${{ steps.find-self.outputs.path }}, and ${{ steps.determine-repository.outputs.repository }} inside run: shell commands. Example: `if [ -n "${{ inputs.action_repository }}" ]; then echo '${{ inputs.action_repository }}'`.

Locations:

- `actions/instrument/deploy/action.yml:115`

### script-injection (severity: high)

Sub-rule (a): The 'Determine version' step directly interpolates ${{ inputs.action_version }}, ${{ steps.find-self.outputs.path }}, and ${{ steps.determine-repository.outputs.repository }} inside run: shell commands. Example: `if [ "${{ inputs.action_version }}" = same ]; then`.

Locations:

- `actions/instrument/deploy/action.yml:128`

### script-injection (severity: high)

Sub-rule (a): The 'Canonicalize' step directly interpolates ${{ inputs.workflows_directory }} and ${{ steps.determine-repository.outputs.repository }} inside run: shell commands, including in a sed substitution: `sed -i 's~'"$repository"'~${{ steps.determine-repository.outputs.repository }}~g' "$workflow_file"`.

Locations:

- `actions/instrument/deploy/action.yml:136`

### script-injection (severity: high)

Sub-rule (a): The 'Deploy workflow-level startup optimization' step directly interpolates ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} inside run: shell commands, including in a curl Authorization header: `--header "Authorization: Bearer ${{ inputs.github_token }}"`.

Locations:

- `actions/instrument/deploy/action.yml:175`

### script-injection (severity: high)

Sub-rule (a): The 'Update workflow-level observability triggers' step directly interpolates ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.workflows_directory }}, ${{ inputs.github_token }}, and ${{ github.repository }} inside run: shell commands. Example: `[ -n "${{ inputs.workflow_level_instrumentation_exclude }}" ] && grep -qvF "$(echo "${{ inputs.workflow_level_instrumentation_exclude }}" | tr ',' '\n' ...)`.

Locations:

- `actions/instrument/deploy/action.yml:195`

### script-injection (severity: high)

Sub-rule (a): The 'Deploy check suite-level instrumentation' step directly interpolates ${{ inputs.workflows_directory }}, ${{ inputs.check_suite_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, and ${{ steps.determine-instrumentation-version.outputs.version }} inside run: shell commands.

Locations:

- `actions/instrument/deploy/action.yml:230`

### script-injection (severity: high)

Sub-rule (a): The 'Deploy job-level instrumentation' step directly interpolates ${{ inputs.workflows_directory }}, ${{ inputs.job_level_instrumentation_exclude }}, ${{ steps.determine-repository.outputs.repository }}, and ${{ steps.determine-instrumentation-version.outputs.version }} inside run: shell commands.

Locations:

- `actions/instrument/deploy/action.yml:290`

### script-injection (severity: high)

Sub-rule (a): The 'Configure job-level instrumentation secret redaction' step directly interpolates ${{ inputs.job_level_instrumentation_secret_redaction_strategy }} and ${{ steps.determine-repository.outputs.repository }} inside run: shell commands, including in a case statement: `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"`.

Locations:

- `actions/instrument/deploy/action.yml:320`

### script-injection (severity: high)

Sub-rule (a): The 'Push' step directly interpolates ${{ inputs.commit_message }} inside a run: shell command: `git commit -m "${{ inputs.commit_message }}"`  A newline or shell metacharacter in the commit message could inject arbitrary commands.

Locations:

- `actions/instrument/deploy/action.yml:430`

### script-injection (severity: high)

Sub-rule (a): The 'Enable auto-merge' step directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} inside a run: shell command: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. A malicious value in this step output could inject shell commands.

Locations:

- `actions/instrument/deploy/action.yml:455`

### github-env-injection (severity: high)

The 'Find self' step writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). A newline embedded in these values could inject arbitrary output variable names/values. Example: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`.

Locations:

- `actions/instrument/deploy/action.yml:104`

### github-env-injection (severity: high)

The 'Determine repository' step writes a value derived from ${{ inputs.action_repository }} to $GITHUB_OUTPUT via `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` without sanitization. A newline in the input could inject arbitrary output variables.

Locations:

- `actions/instrument/deploy/action.yml:125`

### github-env-injection (severity: high)

The 'Determine version' step writes a value derived from ${{ inputs.action_version }} to $GITHUB_OUTPUT via `xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"` without sanitization. A newline in the input could inject arbitrary output variables.

Locations:

- `actions/instrument/deploy/action.yml:133`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 14 findings in actions/instrument/deploy/action.yml by moving every ${{ }} expression out of run: shell commands into env: blocks. Specifically:
1. 'Find self' step: moved inputs.__repository_level_instrumentation_file_name_override, github.workflow, and inputs.workflows_directory to env: block; added printf/tr sanitization before writing to $GITHUB_OUTPUT.
2. 'Determine repository' step: moved inputs.action_repository, steps.find-self.outputs.path, github.action_repository to env: block; sanitized output before writing to $GITHUB_OUTPUT.
3. 'Determine version' step: moved inputs.action_version, steps.find-self.outputs.path, steps.determine-repository.outputs.repository to env: block; sanitized output before writing to $GITHUB_OUTPUT.
4. 'Canonicalize' step: moved inputs.workflows_directory and steps.determine-repository.outputs.repository to env: block; sed substitution now uses $DETERMINE_REPOSITORY env var.
5. 'Deploy workflow-level startup optimization' step: moved steps.determine-instrumentation-version.outputs.version, github.repository_owner, and inputs.github_token to env: block; curl Authorization header now uses $INPUT_GITHUB_TOKEN.
6. 'Update workflow-level observability triggers' step: moved inputs.workflow_level_instrumentation_exclude, inputs.workflows_directory, inputs.github_token, and github.repository to env: block.
7. 'Deploy check suite-level instrumentation' step: moved inputs.workflows_directory, inputs.check_suite_instrumentation_file_name, steps.determine-repository.outputs.repository, and steps.determine-instrumentation-version.outputs.version to env: block.
8. 'Deploy job-level instrumentation' step: moved inputs.workflows_directory, inputs.job_level_instrumentation_exclude, steps.determine-repository.outputs.repository, and steps.determine-instrumentation-version.outputs.version to env: block.
9. 'Configure job-level instrumentation secret redaction' step: moved inputs.job_level_instrumentation_secret_redaction_strategy and steps.determine-repository.outputs.repository to env: block.
10. 'Push' step: moved inputs.commit_message to env: block; git commit now uses $INPUT_COMMIT_MESSAGE.
11. 'Enable auto-merge' step: moved steps.open-pr.outputs.pull-request-number to env: block as PR_NUMBER; gh pr merge now uses "$PR_NUMBER".

