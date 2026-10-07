<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.61.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.61.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in `actions/instrument/deploy-local/action.yml` directly interpolate `${{ }}` expressions — including `${{ inputs.__repository_level_instrumentation_file_name_override }}`, `${{ github.workflow }}`, `${{ inputs.action_version }}`, `${{ inputs.action_repository }}`, `${{ steps.find-self.outputs.path }}`, `${{ steps.determine-repository.outputs.repository }}`, `${{ steps.determine-instrumentation-version.outputs.version }}`, `${{ inputs.workflows_directory }}`, `${{ inputs.workflow_level_instrumentation_workflow_name }}`, `${{ inputs.workflow_level_instrumentation_exclude }}`, `${{ inputs.github_token }}`, `${{ github.repository }}`, `${{ github.repository_owner }}`, `${{ inputs.job_level_instrumentation_exclude }}`, `${{ inputs.job_level_instrumentation_secret_redaction_strategy }}`, `${{ steps.determine-instrumentation-version-comment.outputs.comment }}` — directly into shell command strings. Any of these values can contain shell metacharacters that will be interpreted by bash before the script runs, enabling command injection.

Locations:

- `actions/instrument/deploy-local/action.yml:80`
- `actions/instrument/deploy-local/action.yml:93`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:116`
- `actions/instrument/deploy-local/action.yml:124`
- `actions/instrument/deploy-local/action.yml:132`
- `actions/instrument/deploy-local/action.yml:138`
- `actions/instrument/deploy-local/action.yml:144`
- `actions/instrument/deploy-local/action.yml:151`
- `actions/instrument/deploy-local/action.yml:165`
- `actions/instrument/deploy-local/action.yml:181`
- `actions/instrument/deploy-local/action.yml:215`
- `actions/instrument/deploy-local/action.yml:232`
- `actions/instrument/deploy-local/action.yml:248`
- `actions/instrument/deploy-local/action.yml:265`
- `actions/instrument/deploy-local/action.yml:283`
- `actions/instrument/deploy-local/action.yml:291`
- `actions/instrument/deploy-local/action.yml:315`
- `actions/instrument/deploy-local/action.yml:355`
- `actions/instrument/deploy-local/action.yml:385`

### script-injection (severity: high)

Sub-rule (a): Two `run:` blocks in `actions/instrument/deploy/action.yml` directly interpolate `${{ }}` expressions into shell commands. (1) The 'Push' step uses `git commit -m "${{ inputs.commit_message }}"` — a caller-controlled commit message is interpolated directly into the shell command, enabling command injection via shell metacharacters. (2) The 'Enable auto-merge' step uses `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` — a step output is interpolated unquoted into the shell command.

Locations:

- `actions/instrument/deploy/action.yml:160`
- `actions/instrument/deploy/action.yml:171`

### github-env-injection (severity: high)

Multiple `run:` blocks in `actions/instrument/deploy-local/action.yml` write values derived from untrusted inputs directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). Specifically: (1) The 'Find self' step writes `${{ inputs.__repository_level_instrumentation_file_name_override }}` and `${{ github.workflow }}` directly to `$GITHUB_OUTPUT` via `echo path="${{ ... }}" >> "$GITHUB_OUTPUT"`. (2) The 'Determine repository' step writes a value derived from `${{ inputs.action_repository }}` and `${{ steps.find-self.outputs.path }}` to `$GITHUB_OUTPUT`. (3) The 'Determine version' step writes a value derived from `${{ inputs.action_version }}` and `${{ steps.find-self.outputs.path }}` to `$GITHUB_OUTPUT`. (4) The 'Determine version comment' step writes a value derived from `${{ inputs.action_version }}` to `$GITHUB_OUTPUT`. An attacker who controls any of these inputs can inject newlines to set arbitrary output variables.

Locations:

- `actions/instrument/deploy-local/action.yml:80`
- `actions/instrument/deploy-local/action.yml:93`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:116`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across two files:

1. actions/instrument/deploy/action.yml: Moved `${{ inputs.commit_message }}` to env var `INPUT_COMMIT_MESSAGE` in the Push step, and moved `${{ steps.open-pr.outputs.pull-request-number }}` to env var `PR_NUMBER` in the Enable auto-merge step.

2. actions/instrument/deploy-local/action.yml: Moved all ${{ }} expressions from run: blocks into env: blocks across all 20+ affected steps. For the github-env-injection findings (Find self, Determine repository, Determine version, Determine version comment steps), also added sanitization using `printf '%s' ... | tr -d '\n\r'` before writing to $GITHUB_OUTPUT to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe pipe-to-shell pattern in hardened/action/actions/instrument/shared/install.sh line 26. Replaced `wget -O - https://raw.githubusercontent.com/plengauer/Thoth/main/INSTALL.sh | sh -e` with a safe pattern: download to a temp file via `mktemp` + `wget -O "$_install_script"`, execute with `sh -e "$_install_script"`, then clean up with `rm -f "$_install_script"`. The errexit flag (`-e`) is preserved. No `--` was present to drop.

