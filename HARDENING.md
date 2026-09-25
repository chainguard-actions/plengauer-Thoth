<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.62.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.62.1** was hardened automatically. 22 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): The 'Push' step directly interpolates ${{ inputs.commit_message }} inside a shell run: command: `git commit -m "${{ inputs.commit_message }}"`. This allows an attacker who controls the input to inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy/action.yml:183`

### script-injection (severity: high)

Rule (a): The 'Enable auto-merge' step directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} inside a shell run: command: `run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. Step outputs are workflow-controllable and must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy/action.yml:196`

### script-injection (severity: high)

Rule (a): The 'Find self' step directly interpolates ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ inputs.workflows_directory }}, and ${{ github.workflow }} inside shell run: commands (e.g., `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`). Untrusted inputs must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:80`

### script-injection (severity: high)

Rule (a): The 'Determine repository' step directly interpolates ${{ inputs.action_repository }} and ${{ steps.find-self.outputs.path }} inside shell run: commands (e.g., `if [ -n "${{ inputs.action_repository }}" ]`, `cat "${{ steps.find-self.outputs.path }}" | yq ...`). Untrusted inputs and step outputs must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:100`

### script-injection (severity: high)

Rule (a): The 'Determine version' step directly interpolates ${{ inputs.action_version }}, ${{ steps.find-self.outputs.path }}, and ${{ steps.determine-repository.outputs.repository }} inside shell run: commands (e.g., `if [ "${{ inputs.action_version }}" = same ]`, `echo '${{ inputs.action_version }}'`). Untrusted inputs must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:115`

### script-injection (severity: high)

Rule (a): The 'Determine version comment' step directly interpolates ${{ inputs.action_version }} and ${{ steps.find-self.outputs.path }} inside shell run: commands (e.g., `if [ "${{ inputs.action_version }}" = same ]`, `elif echo '${{ inputs.action_version }}' | grep -qE ...`). Untrusted inputs must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:125`

### script-injection (severity: high)

Rule (a): The 'Canonicalize' step directly interpolates ${{ inputs.workflows_directory }} and ${{ steps.determine-repository.outputs.repository }} inside shell run: commands (e.g., `ls "${{ inputs.workflows_directory }}"/*.yaml`, `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`). Untrusted inputs and step outputs must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:135`

### script-injection (severity: high)

Rule (a): The 'Deploy workflow-level observability' step directly interpolates multiple ${{ inputs.* }} and ${{ steps.*.outputs.* }} expressions inside shell run: commands, including ${{ inputs.workflow_level_instrumentation_workflow_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ inputs.workflows_directory }}, and ${{ inputs.workflow_level_instrumentation_file_name }}. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:180`

### script-injection (severity: high)

Rule (a): The 'Deploy workflow-level startup optimization' step directly interpolates ${{ steps.find-workflow-level-instrumentation.outputs.path }}, ${{ inputs.workflows_directory }}, ${{ inputs.workflow_level_instrumentation_file_name }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:210`

### script-injection (severity: high)

Rule (a): The 'Update workflow-level observability triggers' step directly interpolates ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.workflows_directory }}, ${{ inputs.workflow_level_instrumentation_file_name }}, ${{ inputs.github_token }}, ${{ github.repository }}, and multiple ${{ steps.*.outputs.* }} expressions inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:240`

### script-injection (severity: high)

Rule (a): The 'Deploy check suite-level instrumentation' step directly interpolates ${{ inputs.workflows_directory }}, ${{ inputs.check_suite_level_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, and ${{ steps.find-self.outputs.path }} inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:300`

### script-injection (severity: high)

Rule (a): The 'Deploy check suite-level startup optimization' step directly interpolates ${{ steps.find-check-suite-level-instrumentation.outputs.path }}, ${{ inputs.workflows_directory }}, ${{ inputs.check_suite_instrumentation_file_name }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:330`

### script-injection (severity: high)

Rule (a): The 'Deploy repository-level instrumentation' step directly interpolates ${{ inputs.workflows_directory }}, ${{ inputs.repository_level_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, and ${{ steps.find-self.outputs.path }} inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:360`

### script-injection (severity: high)

Rule (a): The 'Deploy repository-level startup optimization' step directly interpolates ${{ steps.find-repository-level-instrumentation.outputs.path }}, ${{ inputs.workflows_directory }}, ${{ inputs.repository_level_instrumentation_file_name }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:390`

### script-injection (severity: high)

Rule (a): The 'Deploy Copilot Setup' step directly interpolates ${{ inputs.workflows_directory }} inside shell run: commands (e.g., `if [ -r "${{ inputs.workflows_directory }}"/copilot-setup-steps.yml ]`, `echo '...' > "${{ inputs.workflows_directory }}"/copilot-setup-steps.yml`). Untrusted inputs must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:425`

### script-injection (severity: high)

Rule (a): The 'Deploy job-level instrumentation' step directly interpolates ${{ inputs.workflows_directory }}, ${{ inputs.job_level_instrumentation_exclude }}, ${{ steps.find-workflow-level-instrumentation.outputs.path }}, ${{ inputs.workflow_level_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, and ${{ steps.find-self.outputs.path }} inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:435`

### script-injection (severity: high)

Rule (a): The 'Configure job-level instrumentation secret redaction' step directly interpolates ${{ inputs.workflows_directory }}, ${{ steps.determine-repository.outputs.repository }}, ${{ inputs.job_level_instrumentation_secret_redaction_strategy }}, and ${{ steps.find-self.outputs.path }} inside shell run: commands. Untrusted values must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:490`

### script-injection (severity: high)

Rule (a): The 'Annotate instrumentation action versions' step directly interpolates ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ steps.determine-instrumentation-version-comment.outputs.comment }}, ${{ inputs.workflows_directory }}, and ${{ steps.determine-repository.outputs.repository }} inside shell run: commands. Step outputs derived from untrusted inputs must not be interpolated directly into shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:615`

### github-env-injection (severity: high)

The 'Find self' step writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied before the write). For example: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. An attacker-controlled newline in these values can inject arbitrary output variables.

Locations:

- `actions/instrument/deploy-local/action.yml:84`

### github-env-injection (severity: high)

The 'Determine repository' step writes a value derived from ${{ inputs.action_repository }} (via `echo '${{ inputs.action_repository }}'`) to $GITHUB_OUTPUT via xargs without sanitization. An attacker-controlled newline in the input can inject arbitrary output variables.

Locations:

- `actions/instrument/deploy-local/action.yml:107`

### github-env-injection (severity: high)

The 'Determine version' step writes a value derived from ${{ inputs.action_version }} (via `echo '${{ inputs.action_version }}'`) to $GITHUB_OUTPUT via xargs without sanitization. An attacker-controlled newline in the input can inject arbitrary output variables.

Locations:

- `actions/instrument/deploy-local/action.yml:120`

### github-env-injection (severity: high)

The 'Determine version comment' step writes a value derived from ${{ inputs.action_version }} (stored in shell variable `$comment`) to $GITHUB_OUTPUT without sanitization: `echo "comment=# $comment" >> "$GITHUB_OUTPUT"`. An attacker-controlled newline in the input can inject arbitrary output variables.

Locations:

- `actions/instrument/deploy-local/action.yml:132`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across two files:

1. actions/instrument/deploy/action.yml:
   - Push step: moved `inputs.commit_message` to env block as COMMIT_MESSAGE, used `"$COMMIT_MESSAGE"` in git commit command
   - Enable auto-merge step: moved `steps.open-pr.outputs.pull-request-number` to env block as PR_NUMBER, used `"$PR_NUMBER"` in gh pr merge command

2. actions/instrument/deploy-local/action.yml (complete rewrite of all affected steps):
   - Find self step: moved all ${{ }} expressions to env block; added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to GITHUB_OUTPUT
   - Determine repository step: moved inputs/steps/github expressions to env block; added `tr -d '\n\r'` in pipeline before xargs to sanitize GITHUB_OUTPUT writes
   - Determine version step: moved expressions to env block; added `tr -d '\n\r'` sanitization before xargs/GITHUB_OUTPUT write
   - Determine version comment step: moved expressions to env block; added `printf '%s' | tr -d '\n\r'` sanitization before GITHUB_OUTPUT write
   - Canonicalize step: moved workflows_directory and repository to env block
   - Find workflow/check-suite/repository-level observability steps: moved workflows_directory to env block
   - Deploy workflow-level observability step: moved all expressions to env block
   - Deploy workflow-level startup optimization step: moved all expressions to env block
   - Update workflow-level observability triggers step: moved all expressions to env block
   - Deploy check suite-level instrumentation step: moved all expressions to env block
   - Deploy check suite-level startup optimization step: moved all expressions to env block
   - Deploy repository-level instrumentation step: moved all expressions to env block
   - Deploy repository-level startup optimization step: moved all expressions to env block
   - Deploy Copilot Setup step: moved workflows_directory to env block
   - Deploy job-level instrumentation step: moved all expressions to env block
   - Configure job-level instrumentation secret redaction step: moved all expressions to env block
   - Annotate instrumentation action versions step: moved all expressions to env block
   - Restore blank lines step: moved workflows_directory to env block

