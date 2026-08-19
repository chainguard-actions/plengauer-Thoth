<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.58.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.58.0** was hardened automatically. 11 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ ... }} expressions are interpolated directly inside run: shell commands in the composite deploy action. This includes inputs.* (e.g. ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ inputs.action_version }}, ${{ inputs.workflows_directory }}, ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.job_level_instrumentation_exclude }}, ${{ inputs.job_level_instrumentation_secret_redaction_strategy }}, ${{ inputs.commit_message }}), github.* (e.g. ${{ github.workflow }}, ${{ github.repository }}, ${{ github.repository_owner }}), and steps.*.outputs.* (e.g. ${{ steps.find-self.outputs.path }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ steps.open-pr.outputs.pull-request-number }}) used directly in run: blocks. Rule (a) violation: any ${{ ... }} in a run: block is a script injection risk.

Locations:

- `actions/instrument/deploy/action.yml:56`
- `actions/instrument/deploy/action.yml:58`
- `actions/instrument/deploy/action.yml:72`
- `actions/instrument/deploy/action.yml:80`
- `actions/instrument/deploy/action.yml:95`
- `actions/instrument/deploy/action.yml:100`
- `actions/instrument/deploy/action.yml:110`
- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:460`

### script-injection (severity: high)

In build.yml, ${{ matrix.architecture }} is interpolated directly inside run: shell commands (e.g. used in echo, docker pull, and curl commands). Rule (a) violation: matrix.* context values are interpolated directly into shell commands without going through an env: variable.

Locations:

- `.github/workflows/build.yml:43`
- `.github/workflows/build.yml:44`
- `.github/workflows/build.yml:46`
- `.github/workflows/build.yml:48`

### script-injection (severity: high)

In test_shell.yml, multiple ${{ ... }} expressions are interpolated directly inside run: shell commands: ${{ matrix.image }} (used as a docker image argument), ${{ matrix.update }}, ${{ matrix.shell }} (used as a shell name argument to bash -c and apt-get install), ${{ matrix.version }} (used in curl URLs and file names), and ${{ github.token }} (used in Authorization headers). Rule (a) violation.

Locations:

- `.github/workflows/test_shell.yml:75`
- `.github/workflows/test_shell.yml:230`
- `.github/workflows/test_shell.yml:340`
- `.github/workflows/test_shell.yml:390`
- `.github/workflows/test_shell.yml:430`
- `.github/workflows/test_shell.yml:460`
- `.github/workflows/test_shell.yml:510`
- `.github/workflows/test_shell.yml:540`
- `.github/workflows/test_shell.yml:570`

### script-injection (severity: high)

In autobackport.yml, ${{ matrix.ref }} is interpolated directly inside run: shell commands (e.g. git describe --tags --abbrev=0 ${{ matrix.ref }}, git log -1 --pretty=%s "${{ matrix.ref }}", git format-patch -1 "${{ matrix.ref }}"). Also ${{ steps.open-pr.outputs.pull-request-number }} is used directly in: run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}. Rule (a) violation.

Locations:

- `.github/workflows/autobackport.yml:65`
- `.github/workflows/autobackport.yml:80`
- `.github/workflows/autobackport.yml:95`
- `.github/workflows/autobackport.yml:100`
- `.github/workflows/autobackport.yml:115`

### script-injection (severity: high)

In publish.yml, ${{ steps.version.outputs.version }}, ${{ github.token }}, ${{ github.actor }}, and ${{ matrix.ref }} are interpolated directly inside run: shell commands (e.g. version="${{ steps.version.outputs.version }}", echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin, and for tag_simple in ... "${{ matrix.ref }}"). Rule (a) violation.

Locations:

- `.github/workflows/publish.yml:80`
- `.github/workflows/publish.yml:83`
- `.github/workflows/publish.yml:84`
- `.github/workflows/publish.yml:90`
- `.github/workflows/publish.yml:110`

### script-injection (severity: high)

In refresh_demos.yml, ${{ github.token }} and ${{ matrix.demo_directory }} are interpolated directly inside run: shell commands (e.g. wget --header "Authorization: Bearer ${{ github.token }}", export GITHUB_TOKEN=${{ github.token }}, cd demos/${{ matrix.demo_directory }}, sed -i s/${{ github.token }}/***/, git add ... and cd demos/${{ matrix.demo_directory }}). Rule (a) violation.

Locations:

- `.github/workflows/refresh_demos.yml:40`
- `.github/workflows/refresh_demos.yml:50`
- `.github/workflows/refresh_demos.yml:55`
- `.github/workflows/refresh_demos.yml:95`
- `.github/workflows/refresh_demos.yml:100`

### script-injection (severity: high)

In init_fork.yml, ${{ secrets.ACTIONS_GITHUB_TOKEN }} is interpolated directly inside run: shell commands (e.g. [ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ] and curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...). Rule (a) violation.

Locations:

- `.github/workflows/init_fork.yml:18`
- `.github/workflows/init_fork.yml:21`
- `.github/workflows/init_fork.yml:22`

### script-injection (severity: high)

In test_package_repositories.yml, ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} are interpolated directly inside run: shell commands (e.g. echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main" and --header "Authorization: Bearer ${{ github.token }}"). Rule (a) violation.

Locations:

- `.github/workflows/test_package_repositories.yml:22`
- `.github/workflows/test_package_repositories.yml:23`
- `.github/workflows/test_package_repositories.yml:33`

### script-injection (severity: high)

In recompile_agentic_workflows.yml, ${{ steps.open-pr.outputs.pull-request-number }} is interpolated directly inside a run: shell command: run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}. Rule (a) violation.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:55`

### script-injection (severity: high)

In renovate.yml, ${{ steps.open-pr.outputs.pull-request-number }} is interpolated directly inside run: shell commands in multiple jobs (renovate-package-dependency-python, renovate-test-images, renovate-license): run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}. Rule (a) violation.

Locations:

- `.github/workflows/renovate.yml:110`
- `.github/workflows/renovate.yml:230`
- `.github/workflows/renovate.yml:330`

### github-env-injection (severity: high)

In the 'Find self' step of the deploy composite action, ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} are written directly to $GITHUB_OUTPUT without sanitization (no printf '%s' ... | tr -d '\n\r' applied before the write). Example: echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT" and echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT". In the 'Determine version' step, ${{ inputs.action_version }} is also piped to GITHUB_OUTPUT via xargs without sanitization.

Locations:

- `actions/instrument/deploy/action.yml:56`
- `actions/instrument/deploy/action.yml:58`
- `actions/instrument/deploy/action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings across 9 files:

1. actions/instrument/deploy/action.yml: Moved all ${{ }} expressions from run: blocks to env: blocks across all steps (Find self, Determine repository, Determine version, Canonicalize, Find workflow/check-suite/repository-level observability, Deploy workflow/check-suite/repository-level observability and startup optimization, Update workflow-level triggers, Deploy Copilot Setup, Deploy job-level instrumentation, Configure secret redaction, Modify permissions, Restore blank lines, Push, Enable auto-merge). Added sanitization (printf '%s' ... | tr -d '\n\r') before writing to GITHUB_OUTPUT in the Find self step to fix github-env-injection.

2. .github/workflows/build.yml: Moved ${{ matrix.architecture }} to MATRIX_ARCHITECTURE env var in the id:build step.

3. .github/workflows/test_shell.yml: Moved ${{ github.token }}, ${{ matrix.version }}, ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }} to env vars in the upgrade, system-upgrade, dependency, install-manual-os, linux-shell, windows-shell, and performance steps.

4. .github/workflows/autobackport.yml: Moved ${{ matrix.ref }} to MATRIX_REF env var in prepare_branch, format-patch, and prepare_pr steps. Moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var. Also fixed ${{ github.sha }} in setup step.

5. .github/workflows/publish.yml: Moved ${{ steps.version.outputs.version }}, ${{ github.token }}, ${{ github.actor }}, ${{ matrix.ref }} to env vars in the publish step. Also fixed ${{ github.token }} in the setup step.

6. .github/workflows/refresh_demos.yml: Moved ${{ github.token }} and ${{ matrix.demo_directory }} to env vars in the generate job steps.

7. .github/workflows/init_fork.yml: Moved ${{ secrets.ACTIONS_GITHUB_TOKEN }} to ACTIONS_GITHUB_TOKEN_VALUE env var.

8. .github/workflows/test_package_repositories.yml: Moved ${{ github.repository_owner }}, ${{ github.event.repository.name }}, ${{ github.token }} to env vars.

9. .github/workflows/recompile_agentic_workflows.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var.

10. .github/workflows/renovate.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var in all three jobs (renovate-package-dependency-python, renovate-test-images, renovate-license).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed two security findings:

1. github-env-injection in actions/instrument/deploy/action.yml (lines 75, 92): Replaced `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` and `xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"` with `{ read -r _var; safe=$(printf '%s' "$_var" | tr -d '\n\r'); echo "key=$safe" >> "$GITHUB_OUTPUT"; }` to properly sanitize newlines before writing to GITHUB_OUTPUT.

2. script-injection in .github/workflows/test_github.yml (lines 630, 637, 672, 679, 755, 760, 970, 978): Moved all ${{ }} expressions out of run: shell commands and into env: blocks for the affected steps in workflow-smoke, workflow, checksuite-smoke, and deploy jobs. Expressions moved include secrets.GITHUB_TOKEN, secrets.TEST_GITHUB_TOKEN, github.token, github.repository, github.sha, github.workflow, github.ref_name, inputs.ref, matrix.secret_redaction_strategy, matrix.private, env.REPOSITORY_TEMPLATE, and various step outputs. All are now referenced as plain environment variables in the shell scripts.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 5 security findings across 2 workflow files:
1. build.yml: Moved ${{ steps.determine-minimum-version.outputs.version }} into env var MINIMUM_VERSION to prevent script injection in the build-java-agents job.
2. build.yml: Moved ${{ needs.list-python-versions.outputs.versions }} into env var PYTHON_VERSIONS to prevent script injection in the build-deb job.
3. test_github.yml: Moved ${{ steps.my-step.outputs.foo }} into env var MY_STEP_FOO (with proper quoting) to prevent script injection in job-io-1.
4. test_github.yml: Moved ${{ needs.job-io-1.outputs.foo }} into env var JOB_IO_1_FOO (with proper quoting) to prevent script injection in job-io-2.
5. test_github.yml: Added newline sanitization for GITHUB_WORKFLOW_VALUE and GITHUB_REF_NAME_VALUE using printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_OUTPUT in the deploy job's config step.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in .github/workflows/test_shell.yml at the list-images job. Moved `${{ secrets.DOCKERHUB_USERNAME }}` and `${{ secrets.DOCKERHUB_TOKEN }}` from the `run:` shell command string into an `env:` block on the step. The docker login command now uses `"$DOCKERHUB_USERNAME"` and `"$DOCKERHUB_TOKEN"` as plain environment variable references instead of inline template expressions.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities:
1. hardened/action/.github/workflows/build.yml (line ~314): Moved `${{ matrix.version }}` out of the `run:` shell string into an `env:` block as `MATRIX_VERSION`. The shell command now uses `${MATRIX_VERSION}` safely inside double quotes, preventing injection.
2. hardened/action/.github/workflows/test_shell.yml (line ~449): Replaced `bash -c "cd tests && bash run_tests.sh $MATRIX_SHELL"` (where `$MATRIX_SHELL` was unquoted inside a double-quoted bash -c argument) with `(cd tests && bash run_tests.sh "$MATRIX_SHELL")` — a subshell that properly double-quotes the variable, preventing shell metacharacters in the matrix value from being interpreted as commands.

