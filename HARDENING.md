<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.63.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.63.0** was hardened automatically. 11 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Find self' run: block in deploy-local/action.yml directly interpolates ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} inside shell commands. Any of these values can contain shell metacharacters that will be interpreted by bash before the shell ever sees them as data.

Locations:

- `actions/instrument/deploy-local/action.yml:89`

### script-injection (severity: high)

Sub-rule (a): The 'Determine repository' run: block in deploy-local/action.yml directly interpolates ${{ steps.find-self.outputs.path }} (a prior step output) and ${{ inputs.action_repository }} inside shell commands (e.g. `cat "${{ steps.find-self.outputs.path }}"` and `echo '${{ inputs.action_repository }}'`). These values flow through YAML template substitution before the shell sees them.

Locations:

- `actions/instrument/deploy-local/action.yml:103`

### script-injection (severity: high)

Sub-rule (a): The 'Determine version' run: block in deploy-local/action.yml directly interpolates ${{ inputs.action_version }}, ${{ steps.find-self.outputs.path }}, and ${{ steps.determine-repository.outputs.repository }} inside shell commands (e.g. `if [ "${{ inputs.action_version }}" = same ]` and `cat "${{ steps.find-self.outputs.path }}"`). These are script-injection vectors.

Locations:

- `actions/instrument/deploy-local/action.yml:119`

### script-injection (severity: high)

Sub-rule (a): The 'Determine version comment' run: block in deploy-local/action.yml directly interpolates ${{ inputs.action_version }} and ${{ steps.find-self.outputs.path }} inside shell commands (e.g. `if [ "${{ inputs.action_version }}" = same ]` and `echo '${{ inputs.action_version }}'`). These are script-injection vectors.

Locations:

- `actions/instrument/deploy-local/action.yml:130`

### script-injection (severity: high)

Sub-rule (a): Multiple run: blocks in deploy-local/action.yml (Canonicalize, Find workflow-level observability, Find check-suite-level observability, Find repository-level observability, Deploy workflow-level observability, Deploy workflow-level startup optimization, Update workflow-level observability triggers, Deploy check suite-level instrumentation, Deploy check suite-level startup optimization, Deploy repository-level instrumentation, Deploy repository-level startup optimization, Deploy Copilot Setup, Deploy job-level instrumentation, Configure job-level instrumentation secret redaction, Modify permissions for job-level instrumentations, Annotate instrumentation action versions) all directly interpolate ${{ inputs.workflows_directory }}, ${{ inputs.workflow_level_instrumentation_file_name }}, ${{ inputs.workflow_level_instrumentation_workflow_name }}, ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.job_level_instrumentation_exclude }}, ${{ inputs.job_level_instrumentation_secret_redaction_strategy }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ steps.find-self.outputs.path }}, ${{ steps.find-workflow-level-instrumentation.outputs.path }}, ${{ github.repository_owner }}, ${{ github.repository }}, ${{ inputs.github_token }} and other expressions directly inside shell command strings.

Locations:

- `actions/instrument/deploy-local/action.yml:142`
- `actions/instrument/deploy-local/action.yml:148`
- `actions/instrument/deploy-local/action.yml:153`
- `actions/instrument/deploy-local/action.yml:158`
- `actions/instrument/deploy-local/action.yml:163`
- `actions/instrument/deploy-local/action.yml:185`
- `actions/instrument/deploy-local/action.yml:200`
- `actions/instrument/deploy-local/action.yml:230`
- `actions/instrument/deploy-local/action.yml:255`
- `actions/instrument/deploy-local/action.yml:275`
- `actions/instrument/deploy-local/action.yml:295`
- `actions/instrument/deploy-local/action.yml:315`
- `actions/instrument/deploy-local/action.yml:330`
- `actions/instrument/deploy-local/action.yml:345`
- `actions/instrument/deploy-local/action.yml:380`
- `actions/instrument/deploy-local/action.yml:415`

### script-injection (severity: high)

Sub-rule (a): The 'Push' run: block in deploy/action.yml directly interpolates ${{ inputs.commit_message }} inside a shell command: `git commit -m "${{ inputs.commit_message }}"`. An attacker-controlled commit_message value can inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy/action.yml:143`

### script-injection (severity: high)

Sub-rule (a): The 'Enable auto-merge' run: block in deploy/action.yml directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} inside a shell command: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. Step outputs flow through YAML template substitution before the shell sees them and can contain shell metacharacters.

Locations:

- `actions/instrument/deploy/action.yml:163`

### github-env-injection (severity: high)

The 'Find self' run: block in deploy-local/action.yml writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). For example: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. A newline in these values can inject arbitrary key=value pairs into the output file.

Locations:

- `actions/instrument/deploy-local/action.yml:90`

### github-env-injection (severity: high)

The 'Determine repository' run: block in deploy-local/action.yml writes ${{ inputs.action_repository }} (via `echo '${{ inputs.action_repository }}'` piped through xargs) to $GITHUB_OUTPUT without sanitization: `fi | xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"`. A newline in inputs.action_repository can inject arbitrary key=value pairs into the output file.

Locations:

- `actions/instrument/deploy-local/action.yml:118`

### github-env-injection (severity: high)

The 'Determine version' run: block in deploy-local/action.yml writes ${{ inputs.action_version }} (via `echo '${{ inputs.action_version }}'` piped through xargs) to $GITHUB_OUTPUT without sanitization: `fi | xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"`. A newline in inputs.action_version can inject arbitrary key=value pairs into the output file.

Locations:

- `actions/instrument/deploy-local/action.yml:129`

### github-env-injection (severity: high)

The 'Determine version comment' run: block in deploy-local/action.yml writes a value derived from ${{ inputs.action_version }} to $GITHUB_OUTPUT without sanitization: `[ -z "$comment" ] || echo "comment=# $comment" >> "$GITHUB_OUTPUT"`, where $comment is set from `echo '${{ inputs.action_version }}'`. A newline in inputs.action_version can inject arbitrary key=value pairs into the output file.

Locations:

- `actions/instrument/deploy-local/action.yml:140`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in both deploy/action.yml and deploy-local/action.yml.

In deploy/action.yml:
- 'Push' step: moved inputs.commit_message to env block as COMMIT_MESSAGE
- 'Enable auto-merge' step: moved steps.open-pr.outputs.pull-request-number to env block as PR_NUMBER

In deploy-local/action.yml, all ${{ }} expressions were moved from run: blocks to env: blocks across all affected steps (Find self, Determine repository, Determine version, Determine version comment, Canonicalize, Find workflow-level observability, Find check-suite-level observability, Find repository-level observability, Deploy workflow-level observability, Deploy workflow-level startup optimization, Update workflow-level observability triggers, Deploy check suite-level instrumentation, Deploy check suite-level startup optimization, Deploy repository-level instrumentation, Deploy repository-level startup optimization, Deploy Copilot Setup, Deploy job-level instrumentation, Configure job-level instrumentation secret redaction, Modify permissions for job-level instrumentations, Annotate instrumentation action versions, Restore blank lines).

For github-env-injection findings, added printf '%s' ... | tr -d '\n\r' sanitization before writing values to $GITHUB_OUTPUT in the Find self, Determine repository, Determine version, and Determine version comment steps.

