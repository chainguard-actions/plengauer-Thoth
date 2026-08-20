<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.61.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.61.1** was hardened automatically. 9 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in actions/instrument/deploy-local/action.yml. Examples include: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`, `elif [ -r "${{ github.workflow }}" ]`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`, `ls "${{ inputs.workflows_directory }}"/*.yaml`, `if [ -n "${{ inputs.action_repository }}" ]`, `if [ "${{ inputs.action_version }}" = same ]`, `cat "${{ steps.find-self.outputs.path }}"`, `image=ghcr.io/${{ github.repository_owner }}/`, `curl ... --header "Authorization: Bearer ${{ inputs.github_token }}"`, `curl ... /repos/${{ github.repository }}/actions/workflows`, `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`, `version=${{ steps.determine-instrumentation-version.outputs.version }}`, `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"`, and many more. Any of these can allow shell command injection if the input contains shell metacharacters.

Locations:

- `actions/instrument/deploy-local/action.yml:57`
- `actions/instrument/deploy-local/action.yml:58`
- `actions/instrument/deploy-local/action.yml:60`
- `actions/instrument/deploy-local/action.yml:61`
- `actions/instrument/deploy-local/action.yml:64`
- `actions/instrument/deploy-local/action.yml:71`
- `actions/instrument/deploy-local/action.yml:75`
- `actions/instrument/deploy-local/action.yml:76`
- `actions/instrument/deploy-local/action.yml:80`
- `actions/instrument/deploy-local/action.yml:84`
- `actions/instrument/deploy-local/action.yml:88`
- `actions/instrument/deploy-local/action.yml:92`
- `actions/instrument/deploy-local/action.yml:96`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:104`
- `actions/instrument/deploy-local/action.yml:108`
- `actions/instrument/deploy-local/action.yml:112`
- `actions/instrument/deploy-local/action.yml:116`
- `actions/instrument/deploy-local/action.yml:120`
- `actions/instrument/deploy-local/action.yml:124`

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings in actions/instrument/deploy/action.yml. Specifically: `git commit -m "${{ inputs.commit_message }}"` (the commit_message input is interpolated directly into a shell command, allowing injection via crafted commit messages) and `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` (step output interpolated unquoted into shell command).

Locations:

- `actions/instrument/deploy/action.yml:148`
- `actions/instrument/deploy/action.yml:175`

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings in .github/workflows/autobackport.yml. Examples: `git describe --tags --abbrev=0 ${{ github.sha }}`, `cat '${{ github.event_path }}'`, `git format-patch -1 "${{ matrix.ref }}"`, `git cherry-pick -n "${{ matrix.ref }}"`, `git log -1 --pretty=%s "${{ matrix.ref }}"`, `git describe --tags --abbrev=0 ${{ matrix.ref }}`, and `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/autobackport.yml:33`
- `.github/workflows/autobackport.yml:34`
- `.github/workflows/autobackport.yml:60`
- `.github/workflows/autobackport.yml:67`
- `.github/workflows/autobackport.yml:71`
- `.github/workflows/autobackport.yml:77`
- `.github/workflows/autobackport.yml:100`

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings in .github/workflows/publish.yml. Examples: `cat '${{ github.event_path }}'`, `curl -s --fail -H "Authorization: Bearer ${{ github.token }}"`, `version="${{ steps.version.outputs.version }}"`, `echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin`, and `"${{ matrix.ref }}"`.

Locations:

- `.github/workflows/publish.yml:38`
- `.github/workflows/publish.yml:40`
- `.github/workflows/publish.yml:76`
- `.github/workflows/publish.yml:80`
- `.github/workflows/publish.yml:81`
- `.github/workflows/publish.yml:87`

### script-injection (severity: high)

Sub-rule (a): ${{ steps.open-pr.outputs.pull-request-number }} is interpolated directly inside a run: shell command string in .github/workflows/recompile_agentic_workflows.yml: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:52`

### script-injection (severity: high)

Sub-rule (a): ${{ steps.open-pr.outputs.pull-request-number }} is interpolated directly inside run: shell command strings in .github/workflows/renovate.yml: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` (appears in multiple steps).

Locations:

- `.github/workflows/renovate.yml:97`
- `.github/workflows/renovate.yml:197`
- `.github/workflows/renovate.yml:228`

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings in .github/workflows/test_shell.yml. Examples: `bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'` and `sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb`. Matrix values are workflow-controllable and can contain shell metacharacters.

Locations:

- `.github/workflows/test_shell.yml:249`
- `.github/workflows/test_shell.yml:316`
- `.github/workflows/test_shell.yml:317`

### script-injection (severity: high)

Sub-rule (a): ${{ secrets.ACTIONS_GITHUB_TOKEN }} is interpolated directly inside run: shell command strings in .github/workflows/init_fork.yml: `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]` and `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}"`. Any ${{ }} expression in a run: block is a script-injection risk.

Locations:

- `.github/workflows/init_fork.yml:18`
- `.github/workflows/init_fork.yml:20`
- `.github/workflows/init_fork.yml:22`

### script-injection (severity: high)

Sub-rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings in .github/workflows/test_package_repositories.yml. Examples: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"` and `curl ... --header "Authorization: Bearer ${{ github.token }}"`.

Locations:

- `.github/workflows/test_package_repositories.yml:18`
- `.github/workflows/test_package_repositories.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities across 8 files by moving all ${{ }} expressions from run: shell command strings into env: blocks:

1. **actions/instrument/deploy-local/action.yml**: Rewrote the entire file, adding env: blocks to all 15+ steps that had ${{ }} expressions in run: blocks. Key fixes include: inputs.__repository_level_instrumentation_file_name_override, github.workflow, inputs.workflows_directory, steps.find-self.outputs.path, inputs.action_repository, inputs.action_version, steps.determine-repository.outputs.repository, steps.determine-instrumentation-version.outputs.version, github.repository_owner, inputs.github_token, github.repository, inputs.job_level_instrumentation_secret_redaction_strategy, and many more.

2. **actions/instrument/deploy/action.yml**: Fixed two injections: (a) git commit -m "${{ inputs.commit_message }}" → moved commit_message to COMMIT_MESSAGE env var; (b) gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }} → moved to PR_NUMBER env var.

3. **.github/workflows/autobackport.yml**: Fixed 7 injections: github.sha and github.event_path in dynamic step; matrix.ref in prepare_branch, cherry-pick, and prepare_pr steps; gh pr merge with pull-request-number.

4. **.github/workflows/publish.yml**: Fixed 6 injections: github.event_path and github.token in setup/dynamic step; steps.version.outputs.version, github.token, github.actor, and matrix.ref in publish steps.

5. **.github/workflows/recompile_agentic_workflows.yml**: Fixed gh pr merge with pull-request-number.

6. **.github/workflows/renovate.yml**: Fixed 3 occurrences of gh pr merge with pull-request-number (in renovate-package-dependency-python, renovate-test-images, and renovate-license jobs).

7. **.github/workflows/test_shell.yml**: Fixed 3 injections: matrix.image/update/shell in linux-shell job; matrix.version and github.token in performance job (wget, curl, apt-get install, and file rename steps).

8. **.github/workflows/init_fork.yml**: Fixed 2 injections: secrets.ACTIONS_GITHUB_TOKEN in the token check and curl commands.

9. **.github/workflows/test_package_repositories.yml**: Fixed 2 injections: github.repository_owner and github.event.repository.name in apt sources line; github.token in curl command.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 8 findings across 4 files:

1. build.yml (build-http job): Moved matrix.architecture to env block as MATRIX_ARCHITECTURE; sanitized arch output before writing to GITHUB_OUTPUT with tr -d '\n\r'.

2. build.yml (build-python-site-packages job): Moved matrix.version to env block as MATRIX_VERSION for the find -L run step.

3. refresh_demos.yml (generate job): Moved github.token to GITHUB_TOKEN_VAL and matrix.demo_directory to MATRIX_DEMO_DIRECTORY in env blocks; sanitized token before sed substitution.

4. test_shell.yml (upgrade job): Moved github.token to GITHUB_TOKEN_VAL and matrix.version to MATRIX_VERSION in env block.

5. test_shell.yml (system-upgrade, dependency, install-manual-os jobs): Moved matrix.image to MATRIX_IMAGE in env blocks for all three docker run commands.

6. test_shell.yml (windows-shell job): Moved matrix.shell to MATRIX_SHELL in env block; used "$MATRIX_SHELL" in shell commands.

7. deploy-local/action.yml (find-self step): Added printf '%s' | tr -d '\n\r' sanitization for INPUT_FILE_NAME_OVERRIDE and GITHUB_WORKFLOW_VAL before writing to GITHUB_OUTPUT.

8. deploy-local/action.yml (determine-repository step): Refactored to collect value in repo_val variable, then sanitize with tr -d '\n\r' before writing to GITHUB_OUTPUT.

9. deploy-local/action.yml (determine-version step): Refactored to collect value in version_val variable, then sanitize with tr -d '\n\r' before writing to GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 5 findings across 4 files:

1. hardened/action/.github/workflows/build.yml: Moved `${{ needs.list-python-versions.outputs.versions }}` from run: block to env: block as PYTHON_VERSIONS.

2. hardened/action/actions/instrument/deploy-local/action.yml: Added sanitization for the comment variable before writing to $GITHUB_OUTPUT using `printf '%s' "$comment" | tr -d '\n\r'`.

3. hardened/action/.github/workflows/test_shell.yml: Moved DOCKERHUB_USERNAME and DOCKERHUB_TOKEN secrets from run: block to env: block.

4. hardened/action/.github/workflows/test_github.yml (script-injection, 16 locations): Fixed all ${{ }} expressions in run: blocks - GITHUB_TOKEN/TEST_GITHUB_TOKEN in curl Authorization headers moved to env vars; steps.workflow_run.outputs and steps.check_suite.outputs in assertions moved to env vars; INPUT_GITHUB_TOKEN=${{ secrets.GITHUB_TOKEN }} moved to env block; matrix.private, matrix.secret_redaction_strategy, github.repository, inputs.ref, github.token in run blocks moved to env vars; steps.my-step.outputs.foo and needs.job-io-1.outputs.foo moved to env vars; git config user values moved to env vars.

5. hardened/action/.github/workflows/test_github.yml (github-env-injection): Fixed the config step to sanitize the repository name (combining REPOSITORY_TEMPLATE, github.workflow, github.ref_name) before writing to $GITHUB_OUTPUT using `tr -d '\n\r'`.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed three github-env-injection vulnerabilities:
1. hardened/action/.github/workflows/agentics-maintenance.yml (GH_AW_OPERATION): Changed `echo "operation=$GH_AW_OPERATION" >> "$GITHUB_OUTPUT"` to sanitize via `safe_operation=$(printf '%s' "$GH_AW_OPERATION" | tr -d '\n\r')` before writing.
2. hardened/action/.github/workflows/agentics-maintenance.yml (GH_AW_RUN_URL): Changed `echo "run_url=$GH_AW_RUN_URL" >> "$GITHUB_OUTPUT"` to sanitize via `safe_run_url=$(printf '%s' "$GH_AW_RUN_URL" | tr -d '\n\r')` before writing.
3. hardened/action/.github/workflows/autobackport.yml (prepare_pr step): Changed git log output writes to pipe each value through `tr -d '\n\r'` before writing commit_title, author_name, and author_email to GITHUB_OUTPUT, preventing newline injection from attacker-controlled commit metadata.

