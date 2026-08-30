<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.61.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.61.4** was hardened automatically. 13 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings in the deploy-local composite action. Examples include: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"` (Find self step), `if [ -n "${{ inputs.action_repository }}" ]` (Determine repository step), `if [ "${{ inputs.action_version }}" = same ]` (Determine version step), `comment='${{ inputs.action_version }}'` (Determine version comment step), and many more throughout the file. Attacker-controlled inputs are interpolated directly into shell commands before the shell parses them.

Locations:

- `actions/instrument/deploy-local/action.yml:75`
- `actions/instrument/deploy-local/action.yml:88`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:115`
- `actions/instrument/deploy-local/action.yml:130`

### script-injection (severity: high)

Sub-rule (a): In the deploy composite action, `git commit -m "${{ inputs.commit_message }}"` directly interpolates the user-controlled `inputs.commit_message` into a shell command (Push step), and `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` interpolates a step output directly into a shell command (Enable auto-merge step).

Locations:

- `actions/instrument/deploy/action.yml:175`
- `actions/instrument/deploy/action.yml:196`

### script-injection (severity: high)

Sub-rule (a): In autobackport.yml, multiple ${{ ... }} expressions are directly interpolated in run: blocks: `git describe --tags --abbrev=0 ${{ github.sha }}` (setup job, dynamic step), `cat '${{ github.event_path }}'` (setup job), `git describe --tags --abbrev=0 ${{ matrix.ref }}` (backport job, prepare_branch step), `git format-patch -1 "${{ matrix.ref }}"` and `git cherry-pick -n "${{ matrix.ref }}"` (backport job), `git log -1 --pretty=%s "${{ matrix.ref }}"` and `git log -1 --pretty=%an "${{ matrix.ref }}"` (backport job, prepare_pr step), and `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/autobackport.yml:36`
- `.github/workflows/autobackport.yml:38`
- `.github/workflows/autobackport.yml:75`
- `.github/workflows/autobackport.yml:90`
- `.github/workflows/autobackport.yml:100`
- `.github/workflows/autobackport.yml:110`
- `.github/workflows/autobackport.yml:130`

### script-injection (severity: high)

Sub-rule (a): In publish.yml, `echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin` directly interpolates github.token and github.actor into a shell command. Also `version="${{ steps.version.outputs.version }}"` and `for tag_simple in ... "${{ matrix.ref }}"` interpolate step outputs and matrix values into shell commands.

Locations:

- `.github/workflows/publish.yml:85`
- `.github/workflows/publish.yml:83`
- `.github/workflows/publish.yml:92`

### script-injection (severity: high)

Sub-rule (a): In init_fork.yml, `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]` and `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...` directly interpolate secrets into shell commands in run: blocks.

Locations:

- `.github/workflows/init_fork.yml:18`
- `.github/workflows/init_fork.yml:20`

### script-injection (severity: high)

Sub-rule (a): In test_package_repositories.yml, `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"` and `curl ... --header "Authorization: Bearer ${{ github.token }}"` directly interpolate github context values into shell commands.

Locations:

- `.github/workflows/test_package_repositories.yml:18`
- `.github/workflows/test_package_repositories.yml:24`

### script-injection (severity: high)

Sub-rule (a): In refresh_demos.yml, `sed -i s/${{ github.token }}/***/g otlp.json` and `export GITHUB_TOKEN=${{ github.token }}` and `cd demos/${{ matrix.demo_directory }}` directly interpolate github.token and matrix values into shell commands.

Locations:

- `.github/workflows/refresh_demos.yml:60`
- `.github/workflows/refresh_demos.yml:62`
- `.github/workflows/refresh_demos.yml:64`

### script-injection (severity: high)

Sub-rule (a): In test_shell.yml, `run: bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'` directly interpolates matrix values into a shell command. Also `sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb` interpolates matrix.version into a shell command.

Locations:

- `.github/workflows/test_shell.yml:230`
- `.github/workflows/test_shell.yml:310`

### script-injection (severity: high)

Sub-rule (a): In build.yml, `debian_architecture="$(echo ${{ matrix.architecture }} | cut -d / -f 1 | sed 's/le$/el/g')"` and `sudo docker pull --platform linux/${{ matrix.architecture }} docker.io/"$(echo ${{ matrix.architecture }} | tr -d /)"` directly interpolate matrix.architecture into shell commands.

Locations:

- `.github/workflows/build.yml:45`
- `.github/workflows/build.yml:47`

### script-injection (severity: high)

Sub-rule (a): In renovate.yml, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` directly interpolates a step output into a shell command. This pattern appears in three separate jobs.

Locations:

- `.github/workflows/renovate.yml:95`
- `.github/workflows/renovate.yml:160`
- `.github/workflows/renovate.yml:195`

### script-injection (severity: high)

Sub-rule (a): In recompile_agentic_workflows.yml, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` directly interpolates a step output into a shell command.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:55`

### github-env-injection (severity: high)

In the deploy-local composite action, multiple steps write untrusted input values to $GITHUB_OUTPUT without sanitization: (1) 'Find self' step writes `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"` without tr -d newlines sanitization. (2) 'Determine repository' step writes `echo '${{ inputs.action_repository }}'` piped to `echo repository='{}' >> "$GITHUB_OUTPUT"` without sanitization. (3) 'Determine version' step writes `echo '${{ inputs.action_version }}'` piped to `echo version='{}' >> "$GITHUB_OUTPUT"` without sanitization. (4) 'Determine version comment' step writes `echo "comment=# $comment" >> "$GITHUB_OUTPUT"` where comment is derived from `${{ inputs.action_version }}` without sanitization.

Locations:

- `actions/instrument/deploy-local/action.yml:75`
- `actions/instrument/deploy-local/action.yml:77`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:115`
- `actions/instrument/deploy-local/action.yml:130`

### github-env-injection (severity: high)

In autobackport.yml, the 'prepare_pr' step writes git log output derived from ${{ matrix.ref }} to $GITHUB_OUTPUT without sanitization: `echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")"`, `echo author_name="$(git log -1 --pretty=%an "${{ matrix.ref }}")"`, and `echo author_email="$(git log -1 --pretty=%ae "${{ matrix.ref }}")"` are all written directly to $GITHUB_OUTPUT. Commit messages and author names can contain newlines, enabling injection of arbitrary key=value pairs into the output file.

Locations:

- `.github/workflows/autobackport.yml:110`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 9 files:

1. actions/instrument/deploy-local/action.yml: Moved inputs.__repository_level_instrumentation_file_name_override, github.workflow, inputs.workflows_directory, inputs.action_repository, inputs.action_version to env: blocks. Added tr -d '\n\r' sanitization before all GITHUB_OUTPUT writes.

2. actions/instrument/deploy/action.yml: Moved inputs.commit_message to env var in Push step; moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in Enable auto-merge step.

3. .github/workflows/autobackport.yml: Moved github.sha to GITHUB_SHA_VAL env var; replaced cat '${{ github.event_path }}' with cat "$GITHUB_EVENT_PATH"; moved matrix.ref to MATRIX_REF env var in prepare_branch, format-patch/cherry-pick, and prepare_pr steps; sanitized GITHUB_OUTPUT writes with tr -d '\n\r'; moved step output to PR_NUMBER env var in gh pr merge step.

4. .github/workflows/publish.yml: Replaced cat '${{ github.event_path }}' with cat "$GITHUB_EVENT_PATH"; moved github.token to GITHUB_TOKEN_VAL env var; moved github.actor to GITHUB_ACTOR_VAL, matrix.ref to MATRIX_REF, steps.version.outputs.version to VERSION_OUTPUT env vars in docker login/tag step; fixed is_latest and final git tag steps similarly.

5. .github/workflows/init_fork.yml: Moved secrets.ACTIONS_GITHUB_TOKEN to ACTIONS_GITHUB_TOKEN env var in both run: steps.

6. .github/workflows/test_package_repositories.yml: Moved github.repository_owner and github.event.repository.name to env vars; moved github.token to GITHUB_TOKEN_VAL env var.

7. .github/workflows/refresh_demos.yml: Moved github.token to GITHUB_TOKEN_VAL and matrix.demo_directory to DEMO_DIRECTORY env vars; fixed sed command to use env var.

8. .github/workflows/test_shell.yml: Moved matrix.image, matrix.update, matrix.shell to env vars in linux-shell job; moved matrix.version and github.token to env vars in performance job.

9. .github/workflows/build.yml: Moved matrix.architecture to MATRIX_ARCHITECTURE env var; pre-computed arch_nodash variable.

10. .github/workflows/renovate.yml: Moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in all three gh pr merge steps.

11. .github/workflows/recompile_agentic_workflows.yml: Moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in gh pr merge step.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in hardened/action/actions/instrument/deploy-local/action.yml. Moved all ${{ inputs.* }} and ${{ steps.*.outputs.* }} expressions from run: shell blocks into env: blocks for each affected step. The following env vars were introduced: INPUT_WORKFLOWS_DIRECTORY, INPUT_WORKFLOW_LEVEL_INSTRUMENTATION_FILE_NAME, INPUT_WORKFLOW_LEVEL_INSTRUMENTATION_WORKFLOW_NAME, INPUT_WORKFLOW_LEVEL_INSTRUMENTATION_EXCLUDE, INPUT_CHECK_SUITE_INSTRUMENTATION_FILE_NAME, INPUT_CHECK_SUITE_LEVEL_INSTRUMENTATION_FILE_NAME, INPUT_REPOSITORY_LEVEL_INSTRUMENTATION_FILE_NAME, INPUT_JOB_LEVEL_INSTRUMENTATION_EXCLUDE, INPUT_JOB_LEVEL_INSTRUMENTATION_SECRET_REDACTION_STRATEGY, INPUT_GITHUB_TOKEN, FIND_SELF_PATH, FIND_WORKFLOW_LEVEL_PATH, FIND_CHECK_SUITE_LEVEL_PATH, FIND_REPOSITORY_LEVEL_PATH, DETERMINE_REPOSITORY, DETERMINE_VERSION, DETERMINE_VERSION_COMMENT, GITHUB_REPOSITORY_OWNER_VAL, GITHUB_REPOSITORY_VAL. All run: blocks now reference only $ENV_VAR style variables, eliminating the injection risk.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across four workflow files:

1. **test_shell.yml**:
   - upgrade job: moved `${{ github.token }}` and `${{ matrix.version }}` to env vars (GITHUB_TOKEN_VAL, MATRIX_VERSION)
   - list-images job: moved `${{ secrets.DOCKERHUB_USERNAME }}` and `${{ secrets.DOCKERHUB_TOKEN }}` to env vars
   - system-upgrade job: moved `${{ matrix.image }}` to env var (MATRIX_IMAGE)
   - dependency job: added MATRIX_IMAGE env var alongside existing DEPENDENCY env var
   - install-manual-os job: moved `${{ matrix.image }}` to env var (MATRIX_IMAGE)
   - windows-shell job: moved `${{ matrix.shell }}` to env var (MATRIX_SHELL), fixed comparison and apt-get install commands

2. **build.yml**:
   - build-python-site-packages job: moved `${{ matrix.version }}` to env var (MATRIX_VERSION) in the find command
   - build-java-agents job: moved `${{ steps.determine-minimum-version.outputs.version }}` to env var (JAVA_MIN_VERSION)
   - build-deb job: moved `${{ needs.list-python-versions.outputs.versions }}` to env var (PYTHON_VERSIONS)

3. **refresh_demos.yml**:
   - generate job: moved `${{ matrix.demo_directory }}` to env var (DEMO_DIRECTORY) in the step that used it in a cd command

4. **test_github.yml**:
   - workflow-smoke job: moved `${{ secrets.GITHUB_TOKEN }}` to env var (GITHUB_TOKEN_VAL) in curl commands; moved `${{ steps.workflow_run.outputs.id }}` and `${{ steps.workflow_run.outputs.run_attempt }}` to env vars for the [ -n ] checks
   - workflow job: same fixes as workflow-smoke for the duplicate curl and [ -n ] checks
   - checksuite-smoke job: moved `${{ secrets.GITHUB_TOKEN }}` to env var; moved `${{ steps.check_suite.outputs.id }}` to env var for [ -n ] check
   - job-io-1 job: moved `${{ steps.my-step.outputs.foo }}` to env var (MY_STEP_FOO)
   - job-io-2 job: moved `${{ needs.job-io-1.outputs.foo }}` to env var (JOB_IO_1_FOO)
   - deploy job: moved `${{ secrets.TEST_GITHUB_TOKEN }}`, `${{ github.workflow }}`, `${{ github.ref_name }}`, `${{ steps.config.outputs.user }}`, `${{ steps.config.outputs.repository }}`, `${{ matrix.private }}`, `${{ matrix.secret_redaction_strategy }}`, `${{ github.token }}`, `${{ github.repository }}`, `${{ github.sha }}`, `${{ inputs.ref }}` to env vars across multiple steps; fixed github-env-injection by using `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_OUTPUT

