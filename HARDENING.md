<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.62.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.62.0** was hardened automatically. 3 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings. In the 'Push' step, `git commit -m "${{ inputs.commit_message }}"` allows an attacker-controlled commit message to inject shell metacharacters. In the 'Enable auto-merge' step, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` interpolates a step output directly into the shell command.

Locations:

- `actions/instrument/deploy/action.yml:183`
- `actions/instrument/deploy/action.yml:199`

### script-injection (severity: high)

Sub-rule (a): Pervasive direct ${{ }} expression interpolation inside run: shell command strings throughout the deploy-local composite action. Examples include: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `elif [ -r "${{ github.workflow }}" ]`, `if [ "${{ inputs.action_version }}" = same ]`, `echo '${{ inputs.action_version }}'`, `if [ -n "${{ inputs.action_repository }}" ]`, `version=${{ steps.determine-instrumentation-version.outputs.version }}`, `image=ghcr.io/${{ github.repository_owner }}/"$name":"$version"`, `"Authorization: Bearer ${{ inputs.github_token }}"`, `/repos/${{ github.repository }}/actions/workflows`, `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"`, `version='${{ steps.determine-instrumentation-version.outputs.version }}'`, and many more. Any of these allow injection of shell metacharacters via attacker-controlled inputs or context values.

Locations:

- `actions/instrument/deploy-local/action.yml:72`
- `actions/instrument/deploy-local/action.yml:74`
- `actions/instrument/deploy-local/action.yml:76`
- `actions/instrument/deploy-local/action.yml:78`
- `actions/instrument/deploy-local/action.yml:83`
- `actions/instrument/deploy-local/action.yml:88`
- `actions/instrument/deploy-local/action.yml:91`
- `actions/instrument/deploy-local/action.yml:95`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:113`
- `actions/instrument/deploy-local/action.yml:120`
- `actions/instrument/deploy-local/action.yml:130`
- `actions/instrument/deploy-local/action.yml:135`
- `actions/instrument/deploy-local/action.yml:140`
- `actions/instrument/deploy-local/action.yml:145`
- `actions/instrument/deploy-local/action.yml:155`
- `actions/instrument/deploy-local/action.yml:165`
- `actions/instrument/deploy-local/action.yml:175`
- `actions/instrument/deploy-local/action.yml:185`
- `actions/instrument/deploy-local/action.yml:195`
- `actions/instrument/deploy-local/action.yml:205`
- `actions/instrument/deploy-local/action.yml:215`
- `actions/instrument/deploy-local/action.yml:225`
- `actions/instrument/deploy-local/action.yml:235`
- `actions/instrument/deploy-local/action.yml:245`
- `actions/instrument/deploy-local/action.yml:255`
- `actions/instrument/deploy-local/action.yml:265`
- `actions/instrument/deploy-local/action.yml:275`
- `actions/instrument/deploy-local/action.yml:285`

### github-env-injection (severity: high)

In the 'Find self' step of actions/instrument/deploy-local/action.yml, untrusted input values are written directly to $GITHUB_OUTPUT without sanitization. Specifically: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step, allowing newline injection that could poison subsequent steps reading from GITHUB_OUTPUT.

Locations:

- `actions/instrument/deploy-local/action.yml:73`
- `actions/instrument/deploy-local/action.yml:75`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all three findings across two files:

1. deploy/action.yml - 'Push' step: moved inputs.commit_message to env block as COMMIT_MESSAGE; 'Enable auto-merge' step: moved steps.open-pr.outputs.pull-request-number to env block as PR_NUMBER.

2. deploy-local/action.yml - All 30+ ${{ }} expressions in run: shell command strings have been moved to env: blocks for each step (Find self, Determine repository, Determine version, Determine version comment, Canonicalize, Find workflow/check-suite/repository-level observability, Deploy workflow-level observability, Deploy workflow-level startup optimization, Update workflow-level observability triggers, Deploy check suite-level instrumentation, Deploy check suite-level startup optimization, Deploy repository-level instrumentation, Deploy repository-level startup optimization, Deploy Copilot Setup, Deploy job-level instrumentation, Configure job-level instrumentation secret redaction, Modify permissions for job-level instrumentations, Annotate instrumentation action versions, Restore blank lines).

3. deploy-local/action.yml - 'Find self' step: sanitized GITHUB_OUTPUT writes using printf '%s' "$VAR" | tr -d '\n\r' before writing path values to prevent newline injection.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed three instances of unsanitized writes to $GITHUB_OUTPUT in actions/instrument/deploy-local/action.yml:
1. 'Determine repository' step: Replaced `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` with a read+sanitize pattern: `{ IFS= read -r _repo; safe=$(printf '%s' "$_repo" | tr -d '\n\r'); echo "repository=$safe" >> "$GITHUB_OUTPUT"; }`
2. 'Determine version' step: Replaced `xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"` with `{ IFS= read -r _version; safe=$(printf '%s' "$_version" | tr -d '\n\r'); echo "version=$safe" >> "$GITHUB_OUTPUT"; }`
3. 'Determine version comment' step: Replaced `echo "comment=# $comment" >> "$GITHUB_OUTPUT"` with `{ safe=$(printf '%s' "$comment" | tr -d '\n\r'); echo "comment=# $safe" >> "$GITHUB_OUTPUT"; }` — all three now strip newline/carriage-return characters before writing to GITHUB_OUTPUT, preventing header injection attacks.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed three unsanitized GITHUB_OUTPUT writes in actions/instrument/deploy-local/action.yml. In the 'Find workflow-level observability', 'Find check-suite-level observability', and 'Find repository-level observability' steps, replaced `echo path="$workflow_file" >> "$GITHUB_OUTPUT"` with `safe=$(printf '%s' "$workflow_file" | tr -d '\n\r') && echo "path=$safe" >> "$GITHUB_OUTPUT"`. This matches the sanitization pattern already used in the 'Find self' step and prevents newline injection into the GITHUB_OUTPUT file.

### Iteration 4

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings across 8 workflow files:

1. **publish.yml** (lines 73, 100, 118): Moved ${{ steps.version.outputs.version }}, ${{ github.token }}, ${{ github.actor }}, and ${{ matrix.ref }} from run: blocks into env: blocks (VERSION, DOCKER_TOKEN, DOCKER_ACTOR, MATRIX_REF).

2. **test_shell.yml** (lines 248, 310, 316): Moved ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }} into env: block for linux-shell job; moved ${{ matrix.shell }} into env: block for windows-shell job; moved ${{ github.token }} and ${{ matrix.version }} into env: blocks for performance job.

3. **autobackport.yml** (lines 62, 72, 83, 97): Moved ${{ matrix.ref }} into env: blocks for prepare_branch, format-patch, and prepare_pr steps; moved ${{ steps.open-pr.outputs.pull-request-number }} into env: block for gh pr merge step. Also fixed github-env-injection by sanitizing git log output with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

4. **test_package_repositories.yml** (lines 20, 28): Moved ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} into env: blocks.

5. **init_fork.yml** (lines 14, 17): Moved ${{ secrets.ACTIONS_GITHUB_TOKEN }} into env: block as ACTIONS_TOKEN.

6. **refresh_demos.yml** (lines 57, 80, 87): Moved ${{ github.token }} into env: block as GITHUB_TOKEN_VALUE for the wget step; moved ${{ github.token }} and ${{ matrix.demo_directory }} into env: blocks for the demo step; fixed sed command to use $GITHUB_TOKEN_VALUE; fixed additional cd demos/${{ matrix.demo_directory }} occurrences.

7. **recompile_agentic_workflows.yml** (line 52): Moved ${{ steps.open-pr.outputs.pull-request-number }} into env: block as PR_NUMBER.

8. **renovate.yml** (lines 96, 196, 228): Moved ${{ steps.open-pr.outputs.pull-request-number }} into env: block as PR_NUMBER for all three occurrences (renovate-package-dependency-python, renovate-test-images, renovate-license jobs).

9. **build.yml** (line 38): Moved ${{ matrix.architecture }} into env: block as MATRIX_ARCHITECTURE, pre-computed arch_nodash variable to avoid repeated expression expansion.

