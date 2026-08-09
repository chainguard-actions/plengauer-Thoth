<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.60.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.60.0** was hardened automatically. 8 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: steps in actions/instrument/deploy-local/action.yml directly interpolate ${{ ... }} expressions inside shell commands. Examples include: (a) 'Find self' step: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"` and `ls "${{ inputs.workflows_directory }}"/*.yaml`; (b) 'Determine version' step: `if [ "${{ inputs.action_version }}" = same ]` and `echo '${{ inputs.action_version }}'`; (c) 'Determine version comment' step: `if [ "${{ inputs.action_version }}" = same ]` and `comment='${{ inputs.action_version }}'`; (d) 'Canonicalize' step: `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`; (e) 'Deploy workflow-level startup optimization' step: `image=ghcr.io/${{ github.repository_owner }}/...` and `curl ... /users/${{ github.repository_owner }}/packages/...`; (f) 'Update workflow-level observability triggers' step: `if [ -w "${{ steps.find-workflow-level-instrumentation.outputs.path }}" ]` and `ls "${{ inputs.workflows_directory }}"/*.yaml`; (g) 'Deploy job-level instrumentation' step: multiple ${{ inputs.* }} and ${{ steps.*.outputs.* }} interpolations; (h) 'Configure job-level instrumentation secret redaction' step: `${{ inputs.job_level_instrumentation_secret_redaction_strategy }}` in case statement. All these allow an attacker-controlled value to be interpreted as shell code.

Locations:

- `actions/instrument/deploy-local/action.yml:76`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:110`
- `actions/instrument/deploy-local/action.yml:120`
- `actions/instrument/deploy-local/action.yml:130`
- `actions/instrument/deploy-local/action.yml:200`
- `actions/instrument/deploy-local/action.yml:250`
- `actions/instrument/deploy-local/action.yml:300`
- `actions/instrument/deploy-local/action.yml:400`
- `actions/instrument/deploy-local/action.yml:450`

### script-injection (severity: high)

actions/instrument/deploy/action.yml has two run: steps with direct ${{ ... }} interpolation: (a) 'Push' step: `git commit -m "${{ inputs.commit_message }}"` — the commit_message input is user-controlled and injected directly into a shell command; (b) 'Enable auto-merge' step: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` — the step output is interpolated unquoted into the shell command. Sub-rule (a) applies to both.

Locations:

- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:145`

### script-injection (severity: high)

.github/workflows/autobackport.yml has a run: step that directly interpolates a ${{ steps.*.outputs.* }} expression: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. The step output value is injected unquoted into the shell command (sub-rule a).

Locations:

- `.github/workflows/autobackport.yml:130`

### script-injection (severity: high)

.github/workflows/recompile_agentic_workflows.yml has a run: step that directly interpolates a ${{ steps.*.outputs.* }} expression: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. The step output value is injected unquoted into the shell command (sub-rule a).

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:60`

### script-injection (severity: high)

.github/workflows/renovate.yml has three run: steps that directly interpolate ${{ steps.open-pr.outputs.pull-request-number }} into shell commands: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` (appears in three separate jobs). Sub-rule (a) applies.

Locations:

- `.github/workflows/renovate.yml:120`
- `.github/workflows/renovate.yml:280`
- `.github/workflows/renovate.yml:320`

### script-injection (severity: high)

.github/workflows/test_shell.yml has run: steps that directly interpolate ${{ matrix.* }} expressions into shell commands: (a) `bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'` — matrix values are injected directly into a bash -c string; (b) `run: sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb` and `mv opentelemetry-shell_*_all.deb opentelemetry-shell_${{ matrix.version }}_all.deb` — matrix.version is injected into shell commands. Sub-rule (a) applies.

Locations:

- `.github/workflows/test_shell.yml:390`
- `.github/workflows/test_shell.yml:490`
- `.github/workflows/test_shell.yml:495`

### script-injection (severity: high)

.github/workflows/test_package_repositories.yml has a run: step that directly interpolates ${{ github.repository_owner }} and ${{ github.event.repository.name }} into a shell command: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main" | sudo tee /etc/apt/sources.list.d/otel.list`. These github context values are injected unquoted into the shell command (sub-rule a).

Locations:

- `.github/workflows/test_package_repositories.yml:20`

### github-env-injection (severity: high)

actions/instrument/deploy-local/action.yml writes untrusted input values to $GITHUB_OUTPUT without sanitization in multiple steps: (a) 'Find self' step writes `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"` — both inputs.__repository_level_instrumentation_file_name_override and github.workflow are untrusted and written without `printf '%s' ... | tr -d '\n\r'` sanitization; (b) 'Determine version' step writes `echo version='{}' >> "$GITHUB_OUTPUT"` where the value is derived from `${{ inputs.action_version }}` via xargs — no sanitization applied; (c) 'Determine version comment' step writes `echo "comment=# $comment" >> "$GITHUB_OUTPUT"` where $comment is derived from `${{ inputs.action_version }}` — no sanitization applied.

Locations:

- `actions/instrument/deploy-local/action.yml:76`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:110`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings across 7 files:

1. **actions/instrument/deploy-local/action.yml**: Moved all ${{ }} expressions from run: blocks to env: blocks across all steps: 'Find self', 'Determine repository', 'Determine version', 'Determine version comment', 'Canonicalize', 'Find workflow/check-suite/repository-level observability', 'Deploy workflow/check-suite/repository-level instrumentation and startup optimization', 'Update workflow-level observability triggers', 'Deploy Copilot Setup', 'Deploy job-level instrumentation', 'Configure job-level instrumentation secret redaction', 'Modify permissions for job-level instrumentations', 'Restore blank lines', 'Annotate instrumentation action versions'. Also fixed github-env-injection in 'Find self' and 'Determine version comment' steps by using printf '%s' ... | tr -d '\n\r' before writing to GITHUB_OUTPUT.

2. **actions/instrument/deploy/action.yml**: Moved inputs.commit_message to COMMIT_MESSAGE env var in 'Push' step; moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in 'Enable auto-merge' step.

3. **.github/workflows/autobackport.yml**: Moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in the gh pr merge step.

4. **.github/workflows/recompile_agentic_workflows.yml**: Moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in the gh pr merge step.

5. **.github/workflows/renovate.yml**: Fixed three occurrences of gh pr merge with unquoted step output - moved pull-request-number to PR_NUMBER env var in renovate-package-dependency-python, renovate-test-images, and renovate-license jobs.

6. **.github/workflows/test_shell.yml**: Fixed multiple matrix value injections: moved matrix.image/update/shell to env vars in linux-shell job; moved matrix.version to env vars in performance job (wget, mv, apt-get install, and performance list file steps); moved matrix.image to env var in system-upgrade, dependency, and install-manual-os jobs; moved matrix.shell to env var in windows-shell job.

7. **.github/workflows/test_package_repositories.yml**: Moved github.repository_owner and github.event.repository.name to REPO_OWNER and REPO_NAME env vars in the apt sources.list step.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities across 8 workflow files by moving ${{ }} expressions into env: blocks and referencing them as plain environment variables:

1. **build.yml**: Added `MATRIX_ARCHITECTURE` env var for the build step (replacing 8 occurrences of `${{ matrix.architecture }}`), and added `JAVA_MIN_VERSION` env var for the echo version step (replacing `${{ steps.determine-minimum-version.outputs.version }}`).

2. **autobackport.yml**: Added `GIT_SHA` and `EVENT_PATH` env vars for the setup/dynamic step; added `MATRIX_REF` env var for the prepare_branch, format-patch/cherry-pick, and prepare_pr steps.

3. **publish.yml**: Added `EVENT_PATH` and `GITHUB_TOKEN_VAL` env vars for the setup/dynamic step; added `RELEASE_VERSION`, `GITHUB_TOKEN_VAL`, `GITHUB_ACTOR_VAL`, and `MATRIX_REF` env vars for the docker push step; added `RELEASE_VERSION` to the is_latest step; added `RELEASE_VERSION` to the final git tag step.

4. **refresh_demos.yml**: Added `GITHUB_TOKEN_VAL` env var for the wget step; added `GITHUB_TOKEN_VAL` and `DEMO_DIRECTORY` env vars for the demo step (also fixed `sed -i s/${{ github.token }}/***/g` to use the env var); added `DEMO_DIRECTORY` env var for the second generate step and the diff step.

5. **test_shell.yml**: Added `GITHUB_TOKEN_VAL` and `MATRIX_VERSION` env vars for the upgrade step.

6. **test_github.yml**: Added `TEST_GITHUB_TOKEN`, `GITHUB_TOKEN_VAL`, `GITHUB_REPOSITORY_VAL`, `GITHUB_SHA_VAL`, `CONFIG_USER`, `CONFIG_REPOSITORY`, `SECRET_REDACTION_STRATEGY`, and `INPUTS_REF` env vars for the step containing the printf/wget commands.

7. **test_package_repositories.yml**: Added `GITHUB_TOKEN_VAL` env var for the curl step.

8. **init_fork.yml**: Added `ACTIONS_GITHUB_TOKEN_VAL` env var for both the token check step and the curl/enable workflows step.

The two 'permissions' findings were already noted as passing (they have job-level permissions blocks), so no changes were needed for those.

