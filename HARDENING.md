<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.54.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.54.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `actions/instrument/deploy/action.yml` directly interpolate `${{ }}` expressions inside shell command strings (sub-rule a). This allows an attacker-controlled value to be injected into the shell before quoting can occur. Affected expressions include: `${{ inputs.__repository_level_instrumentation_file_name_override }}` (Find self step), `${{ github.workflow }}` (Find self step), `${{ inputs.action_repository }}` (Determine repository step), `${{ inputs.action_version }}` (Determine version step), `${{ inputs.workflows_directory }}` (multiple steps), `${{ inputs.workflow_level_instrumentation_exclude }}` (Update workflow-level observability triggers step), `${{ inputs.job_level_instrumentation_exclude }}` (Deploy job-level instrumentation step), `${{ inputs.job_level_instrumentation_secret_redaction_strategy }}` (Configure job-level instrumentation secret redaction step), `${{ inputs.commit_message }}` (Push step), `${{ github.repository_owner }}` (multiple startup optimization steps), `${{ github.repository }}` (Update workflow-level observability triggers step), and `${{ steps.*.outputs.* }}` references throughout. All of these should be moved to `env:` variables and then referenced as quoted `"$VAR"` in the shell.

Locations:

- `actions/instrument/deploy/action.yml:93`
- `actions/instrument/deploy/action.yml:95`
- `actions/instrument/deploy/action.yml:97`
- `actions/instrument/deploy/action.yml:103`
- `actions/instrument/deploy/action.yml:108`
- `actions/instrument/deploy/action.yml:113`
- `actions/instrument/deploy/action.yml:120`
- `actions/instrument/deploy/action.yml:127`
- `actions/instrument/deploy/action.yml:133`
- `actions/instrument/deploy/action.yml:140`
- `actions/instrument/deploy/action.yml:148`
- `actions/instrument/deploy/action.yml:160`
- `actions/instrument/deploy/action.yml:175`
- `actions/instrument/deploy/action.yml:195`
- `actions/instrument/deploy/action.yml:220`
- `actions/instrument/deploy/action.yml:260`
- `actions/instrument/deploy/action.yml:310`
- `actions/instrument/deploy/action.yml:360`
- `actions/instrument/deploy/action.yml:410`
- `actions/instrument/deploy/action.yml:460`

### github-env-injection (severity: high)

The `Find self` step in `actions/instrument/deploy/action.yml` writes values derived directly from `${{ inputs.__repository_level_instrumentation_file_name_override }}` and `${{ github.workflow }}` to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). For example: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. Similarly, the `Determine repository` step writes `${{ inputs.action_repository }}` to `$GITHUB_OUTPUT` via xargs without sanitization, and the `Determine version` step writes `${{ inputs.action_version }}` to `$GITHUB_OUTPUT` via xargs without sanitization. A newline character in any of these values could inject arbitrary environment variables or outputs.

Locations:

- `actions/instrument/deploy/action.yml:95`
- `actions/instrument/deploy/action.yml:97`
- `actions/instrument/deploy/action.yml:108`
- `actions/instrument/deploy/action.yml:120`

### unpinned-uses (severity: high)

The `Checkout` step in `actions/instrument/deploy/action.yml` uses `actions/checkout@v6.0.2`, which is a mutable tag reference rather than a pinned 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit, enabling a supply-chain attack. It should be pinned to a full SHA, e.g. `actions/checkout@<40-char-sha> # v6.0.2`.

Locations:

- `actions/instrument/deploy/action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in actions/instrument/deploy/action.yml:

1. unpinned-uses: Pinned actions/checkout@v6.0.2 to full SHA de0fac2e4500dabe0009e67214ff5f5447ce83dd with tag comment.

2. script-injection: Moved all ${{ }} expressions from run: blocks to env: blocks. Key mappings: inputs.__repository_level_instrumentation_file_name_override→FILE_NAME_OVERRIDE, github.workflow→GITHUB_WORKFLOW_NAME, inputs.action_repository→INPUT_ACTION_REPOSITORY, inputs.action_version→INPUT_ACTION_VERSION, inputs.workflows_directory→WORKFLOWS_DIRECTORY, inputs.workflow_level_instrumentation_exclude→WORKFLOW_LEVEL_INSTRUMENTATION_EXCLUDE, inputs.job_level_instrumentation_exclude→JOB_LEVEL_INSTRUMENTATION_EXCLUDE, inputs.job_level_instrumentation_secret_redaction_strategy→JOB_LEVEL_INSTRUMENTATION_SECRET_REDACTION_STRATEGY, inputs.commit_message→COMMIT_MESSAGE, github.repository_owner→GITHUB_REPOSITORY_OWNER_VAR, github.repository→GITHUB_REPOSITORY_VAR, and all steps.*.outputs.* references.

3. github-env-injection: The Find self step now sanitizes values with printf '%s' | tr -d '\n\r' before writing to $GITHUB_OUTPUT. The Determine repository and Determine version steps use a sanitizing sh -c wrapper with printf and tr -d '\n\r' before writing to $GITHUB_OUTPUT.

