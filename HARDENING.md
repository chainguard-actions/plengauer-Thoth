<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.63.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.63.1** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in the 'Find self' step. The expressions ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} are embedded directly in shell commands, allowing an attacker who controls these inputs to inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:98`

### script-injection (severity: high)

Sub-rule (a): In the 'Determine repository' step, ${{ inputs.action_repository }} is interpolated directly inside a run: shell command string (e.g., `if [ -n "${{ inputs.action_repository }}" ]; then echo '${{ inputs.action_repository }}'`). An attacker controlling this input can inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:120`

### script-injection (severity: high)

Sub-rule (a): In the 'Determine version' step, ${{ inputs.action_version }} is interpolated directly inside a run: shell command string (e.g., `if [ "${{ inputs.action_version }}" = same ]` and `echo '${{ inputs.action_version }}'`). An attacker controlling this input can inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:140`

### script-injection (severity: high)

Sub-rule (a): In the 'Determine version comment' step, ${{ inputs.action_version }} and ${{ steps.find-self.outputs.path }} are interpolated directly inside a run: shell command string. An attacker controlling these values can inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:155`

### script-injection (severity: high)

Sub-rule (a): In the 'Deploy workflow-level startup optimization', 'Deploy check suite-level startup optimization', and 'Deploy repository-level startup optimization' steps, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} are interpolated directly inside run: shell command strings (e.g., `version=${{ steps.determine-instrumentation-version.outputs.version }}`, `image=ghcr.io/${{ github.repository_owner }}/...`). These allow injection of arbitrary shell commands.

Locations:

- `actions/instrument/deploy-local/action.yml:175`
- `actions/instrument/deploy-local/action.yml:220`
- `actions/instrument/deploy-local/action.yml:265`

### script-injection (severity: high)

Sub-rule (a): In the 'Push' step, ${{ inputs.commit_message }} is interpolated directly inside a run: shell command string: `git commit -m "${{ inputs.commit_message }}"`. An attacker who controls the commit_message input can inject arbitrary shell commands into the git commit invocation.

Locations:

- `actions/instrument/deploy/action.yml:148`

### script-injection (severity: high)

Sub-rule (a): In the 'Enable auto-merge' step, ${{ steps.open-pr.outputs.pull-request-number }} is interpolated directly inside a run: shell command string: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. A compromised or manipulated step output could inject arbitrary arguments or shell commands.

Locations:

- `actions/instrument/deploy/action.yml:168`

### github-env-injection (severity: high)

In the 'Find self' step, the values ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} are written directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline character in these values could inject additional key=value pairs into the GitHub output context.

Locations:

- `actions/instrument/deploy-local/action.yml:99`
- `actions/instrument/deploy-local/action.yml:101`

### github-env-injection (severity: high)

In the 'Determine version comment' step, a value derived from ${{ inputs.action_version }} (stored in the shell variable $comment) is written to $GITHUB_OUTPUT via `echo "comment=# $comment" >> "$GITHUB_OUTPUT"` without the required sanitization step (printf '%s' ... | tr -d '\n\r'). A newline in the input could inject additional key=value pairs into the GitHub output context.

Locations:

- `actions/instrument/deploy-local/action.yml:162`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across two files:

1. actions/instrument/deploy-local/action.yml:
   - 'Find self' step: Moved ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} into env vars (REPO_LEVEL_OVERRIDE, GITHUB_WORKFLOW_NAME, WORKFLOWS_DIRECTORY). Added printf/tr sanitization before writing to $GITHUB_OUTPUT.
   - 'Determine repository' step: Moved ${{ inputs.action_repository }} into env var INPUT_ACTION_REPOSITORY.
   - 'Determine version' step: Moved ${{ inputs.action_version }} into env var INPUT_ACTION_VERSION.
   - 'Determine version comment' step: Moved ${{ inputs.action_version }} and ${{ steps.find-self.outputs.path }} into env vars. Added printf/tr sanitization before writing $comment to $GITHUB_OUTPUT.
   - 'Deploy workflow-level startup optimization' step: Moved ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ github.repository_owner }}, and ${{ inputs.github_token }} into env vars (INSTRUMENTATION_VERSION, REPOSITORY_OWNER, GITHUB_TOKEN_INPUT).
   - 'Deploy check suite-level startup optimization' step: Same env var fixes.
   - 'Deploy repository-level startup optimization' step: Same env var fixes.

2. actions/instrument/deploy/action.yml:
   - 'Push' step: Moved ${{ inputs.commit_message }} into env var COMMIT_MESSAGE.
   - 'Enable auto-merge' step: Moved ${{ steps.open-pr.outputs.pull-request-number }} into env var PR_NUMBER.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 20 script-injection locations in actions/instrument/deploy-local/action.yml by moving every ${{ }} expression from run: blocks into env: blocks and referencing them as $ENV_VAR_NAME in shell scripts. Fixed both github-env-injection locations in 'Determine repository' and 'Determine version' steps by replacing xargs-based GITHUB_OUTPUT writes with sanitized writes using printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

