<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.61.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.61.6** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in the 'Push' step. Specifically, `git commit -m "${{ inputs.commit_message }}"` injects the user-controlled commit_message input directly into a shell command, enabling command injection. The 'Enable auto-merge' step also uses `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` with an unquoted steps output expression directly in the run block.

Locations:

- `actions/instrument/deploy/action.yml:170`
- `actions/instrument/deploy/action.yml:192`

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings throughout deploy-local/action.yml. The 'Find self' step uses `${{ inputs.__repository_level_instrumentation_file_name_override }}`, `${{ github.workflow }}`, and `${{ inputs.workflows_directory }}` directly in shell commands. The 'Determine repository' step uses `${{ inputs.action_repository }}` and `${{ steps.find-self.outputs.path }}` directly in shell commands. The 'Determine version' step uses `${{ inputs.action_version }}` and `${{ steps.determine-repository.outputs.repository }}` directly in shell commands. The 'Determine version comment' step uses `${{ inputs.action_version }}` directly in shell commands. The 'Deploy workflow-level startup optimization' step uses the unquoted assignment `version=${{ steps.determine-instrumentation-version.outputs.version }}` directly in a shell script. All of these allow an attacker-controlled value to be interpreted as shell syntax before the shell ever sees it.

Locations:

- `actions/instrument/deploy-local/action.yml:75`
- `actions/instrument/deploy-local/action.yml:77`
- `actions/instrument/deploy-local/action.yml:91`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:116`
- `actions/instrument/deploy-local/action.yml:128`
- `actions/instrument/deploy-local/action.yml:243`

### github-env-injection (severity: high)

In the 'Find self' step of deploy-local/action.yml, the values of ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} are written directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). An attacker-controlled newline in these values could inject arbitrary key=value pairs into the GitHub Actions output context, potentially overwriting subsequent step outputs. Specifically: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`.

Locations:

- `actions/instrument/deploy-local/action.yml:76`
- `actions/instrument/deploy-local/action.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings:

1. deploy/action.yml - 'Push' step: Moved inputs.commit_message into env block as COMMIT_MESSAGE; 'Enable auto-merge' step: Moved steps.open-pr.outputs.pull-request-number into env block as PR_NUMBER.

2. deploy-local/action.yml - 'Find self' step: Moved __repository_level_instrumentation_file_name_override, github.workflow, and workflows_directory into env block; 'Determine repository' step: Moved inputs.action_repository and steps.find-self.outputs.path into env block; 'Determine version' step: Moved inputs.action_version, steps.find-self.outputs.path, and steps.determine-repository.outputs.repository into env block; 'Determine version comment' step: Moved inputs.action_version and steps.find-self.outputs.path into env block; 'Deploy workflow-level/check-suite-level/repository-level startup optimization' steps: Moved instrumentation version, repository owner, github token, and path expressions into env blocks.

3. deploy-local/action.yml - 'Find self' step: Added sanitization with printf '%s' "$VAR" | tr -d '\n\r' before writing values to $GITHUB_OUTPUT to prevent newline injection.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection findings by moving every ${{ }} expression from run: blocks into env: blocks, then referencing them as plain environment variables in the shell scripts. Fixed github-env-injection findings in the 'Determine version' and 'Determine version comment' steps by using `printf '%s' "$ACTION_VERSION" | tr -d '\n\r'` to sanitize the ACTION_VERSION value before writing to $GITHUB_OUTPUT. All ${{ }} expressions in if: conditions and env: blocks are acceptable and were left in place.

