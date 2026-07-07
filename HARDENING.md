<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.59.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.59.0** was hardened automatically. 12 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple run: blocks in actions/instrument/deploy/action.yml directly interpolate ${{ ... }} expressions. Examples include: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`, `echo '${{ inputs.action_repository }}'`, `if [ "${{ inputs.action_version }}" = same ]`, `cat "${{ steps.find-self.outputs.path }}"`, `version=${{ steps.determine-instrumentation-version.outputs.version }}` (unquoted), `--header "Authorization: Bearer ${{ inputs.github_token }}"`, `/repos/${{ github.repository }}/actions/workflows`, `ghcr.io/${{ github.repository_owner }}/`, `git commit -m "${{ inputs.commit_message }}"`, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`, `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"`, `[ -n "${{ inputs.workflow_level_instrumentation_exclude }}" ]`, `[ -n "${{ inputs.job_level_instrumentation_exclude }}" ]`, `echo 'name: ${{ inputs.workflow_level_instrumentation_workflow_name }}`, and many more. All of these are direct expression interpolations in shell run: blocks without routing through env: variables.

Locations:

- `actions/instrument/deploy/action.yml:68`
- `actions/instrument/deploy/action.yml:70`
- `actions/instrument/deploy/action.yml:72`
- `actions/instrument/deploy/action.yml:80`
- `actions/instrument/deploy/action.yml:83`
- `actions/instrument/deploy/action.yml:86`
- `actions/instrument/deploy/action.yml:92`
- `actions/instrument/deploy/action.yml:97`
- `actions/instrument/deploy/action.yml:100`
- `actions/instrument/deploy/action.yml:107`
- `actions/instrument/deploy/action.yml:113`
- `actions/instrument/deploy/action.yml:120`
- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:145`
- `actions/instrument/deploy/action.yml:155`
- `actions/instrument/deploy/action.yml:165`
- `actions/instrument/deploy/action.yml:175`
- `actions/instrument/deploy/action.yml:185`
- `actions/instrument/deploy/action.yml:195`
- `actions/instrument/deploy/action.yml:205`
- `actions/instrument/deploy/action.yml:215`
- `actions/instrument/deploy/action.yml:225`
- `actions/instrument/deploy/action.yml:235`
- `actions/instrument/deploy/action.yml:245`
- `actions/instrument/deploy/action.yml:255`
- `actions/instrument/deploy/action.yml:265`
- `actions/instrument/deploy/action.yml:275`
- `actions/instrument/deploy/action.yml:285`
- `actions/instrument/deploy/action.yml:295`
- `actions/instrument/deploy/action.yml:305`
- `actions/instrument/deploy/action.yml:315`
- `actions/instrument/deploy/action.yml:325`
- `actions/instrument/deploy/action.yml:335`
- `actions/instrument/deploy/action.yml:345`
- `actions/instrument/deploy/action.yml:355`
- `actions/instrument/deploy/action.yml:365`
- `actions/instrument/deploy/action.yml:375`
- `actions/instrument/deploy/action.yml:385`
- `actions/instrument/deploy/action.yml:395`
- `actions/instrument/deploy/action.yml:405`
- `actions/instrument/deploy/action.yml:415`
- `actions/instrument/deploy/action.yml:425`
- `actions/instrument/deploy/action.yml:435`
- `actions/instrument/deploy/action.yml:445`
- `actions/instrument/deploy/action.yml:455`
- `actions/instrument/deploy/action.yml:465`
- `actions/instrument/deploy/action.yml:475`
- `actions/instrument/deploy/action.yml:485`
- `actions/instrument/deploy/action.yml:495`
- `actions/instrument/deploy/action.yml:505`

### script-injection (severity: high)

Rule (a): .github/workflows/autobackport.yml directly interpolates ${{ matrix.ref }} and ${{ github.sha }} in multiple run: blocks. Examples: `git format-patch -1 "${{ matrix.ref }}" --stdout > /tmp/commit.patch`, `git cherry-pick -n "${{ matrix.ref }}"`, `echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"`, `git describe --tags --abbrev=0 ${{ github.sha }}`, `cat '${{ github.event_path }}'`, and `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/autobackport.yml:37`
- `.github/workflows/autobackport.yml:55`
- `.github/workflows/autobackport.yml:62`
- `.github/workflows/autobackport.yml:65`
- `.github/workflows/autobackport.yml:70`
- `.github/workflows/autobackport.yml:80`
- `.github/workflows/autobackport.yml:88`
- `.github/workflows/autobackport.yml:95`
- `.github/workflows/autobackport.yml:100`
- `.github/workflows/autobackport.yml:105`

### script-injection (severity: high)

Rule (a): .github/workflows/build.yml directly interpolates ${{ matrix.architecture }} in run: blocks. Examples: `debian_architecture="$(echo ${{ matrix.architecture }} | cut -d / -f 1 | sed 's/le$/el/g')"`, `sudo docker pull --platform linux/${{ matrix.architecture }} docker.io/"$(echo ${{ matrix.architecture }} | tr -d /)"`, `echo "no debian found supporting the architecture ${{ matrix.architecture }}"`.

Locations:

- `.github/workflows/build.yml:45`
- `.github/workflows/build.yml:46`
- `.github/workflows/build.yml:47`
- `.github/workflows/build.yml:48`
- `.github/workflows/build.yml:49`
- `.github/workflows/build.yml:50`
- `.github/workflows/build.yml:51`
- `.github/workflows/build.yml:52`
- `.github/workflows/build.yml:53`

### script-injection (severity: high)

Rule (a): .github/workflows/publish.yml directly interpolates ${{ steps.version.outputs.version }}, ${{ matrix.ref }}, ${{ github.token }}, and ${{ github.actor }} in run: blocks. Examples: `version="${{ steps.version.outputs.version }}"`, `echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin`, `for tag_simple in ... "${{ matrix.ref }}"`.

Locations:

- `.github/workflows/publish.yml:55`
- `.github/workflows/publish.yml:60`
- `.github/workflows/publish.yml:65`
- `.github/workflows/publish.yml:70`

### script-injection (severity: high)

Rule (a): .github/workflows/refresh_demos.yml directly interpolates ${{ github.token }} and ${{ matrix.demo_directory }} in run: blocks. Examples: `export GITHUB_TOKEN=${{ github.token }}`, `cd demos/${{ matrix.demo_directory }}`, `sed -i s/${{ github.token }}/***/g otlp.json`, `curl --header "Authorization: Bearer ${{ github.token }}"`.

Locations:

- `.github/workflows/refresh_demos.yml:50`
- `.github/workflows/refresh_demos.yml:55`
- `.github/workflows/refresh_demos.yml:60`
- `.github/workflows/refresh_demos.yml:65`
- `.github/workflows/refresh_demos.yml:70`

### script-injection (severity: high)

Rule (a): .github/workflows/renovate.yml directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} in multiple run: blocks. Example: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/renovate.yml:100`
- `.github/workflows/renovate.yml:150`
- `.github/workflows/renovate.yml:200`

### script-injection (severity: high)

Rule (a): .github/workflows/recompile_agentic_workflows.yml directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} in a run: block. Example: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:50`

### script-injection (severity: high)

Rule (a): .github/workflows/test_shell.yml directly interpolates ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }}, and ${{ matrix.version }} in run: blocks. Examples: `bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"`, `opentelemetry-shell_${{ matrix.version }}_all.deb`, `wget ... -O opentelemetry-shell_${{ matrix.version }}_all.deb "$url"`, `curl ... https://api.github.com/repos/.../releases/tags/${{ matrix.version }}`.

Locations:

- `.github/workflows/test_shell.yml:200`
- `.github/workflows/test_shell.yml:205`
- `.github/workflows/test_shell.yml:210`
- `.github/workflows/test_shell.yml:215`
- `.github/workflows/test_shell.yml:220`
- `.github/workflows/test_shell.yml:225`
- `.github/workflows/test_shell.yml:230`
- `.github/workflows/test_shell.yml:235`
- `.github/workflows/test_shell.yml:240`

### script-injection (severity: high)

Rule (a): .github/workflows/init_fork.yml directly interpolates ${{ secrets.ACTIONS_GITHUB_TOKEN }} in run: blocks. Examples: `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]`, `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}"`.

Locations:

- `.github/workflows/init_fork.yml:15`
- `.github/workflows/init_fork.yml:18`
- `.github/workflows/init_fork.yml:19`

### script-injection (severity: high)

Rule (a): .github/workflows/test_package_repositories.yml directly interpolates ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} in run: blocks. Examples: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"`, `--header "Authorization: Bearer ${{ github.token }}"`.

Locations:

- `.github/workflows/test_package_repositories.yml:20`
- `.github/workflows/test_package_repositories.yml:25`
- `.github/workflows/test_package_repositories.yml:30`

### github-env-injection (severity: high)

In actions/instrument/deploy/action.yml, the 'Find self' step writes untrusted input values directly to $GITHUB_OUTPUT without sanitization. Specifically: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. Neither value is sanitized with `printf '%s' ... | tr -d '\n\r'` before the write. Similarly, the 'prepare_pr' step in autobackport.yml writes `${{ matrix.ref }}`-derived values (commit_title, author_name, author_email) to $GITHUB_OUTPUT without sanitization.

Locations:

- `actions/instrument/deploy/action.yml:69`
- `actions/instrument/deploy/action.yml:71`
- `.github/workflows/autobackport.yml:80`
- `.github/workflows/autobackport.yml:81`
- `.github/workflows/autobackport.yml:82`

### permissions (severity: medium)

missing-permissions: .github/workflows/copilot-setup-steps.yml has no top-level permissions: key, and the single job 'copilot-setup-steps' has a job-level permissions: block but the file has no top-level permissions. However, the job does have permissions: actions: read at the job level, so this passes. Actually re-checking: the file has no top-level permissions and the job has permissions: actions: read — this PASSES per the check rules since every job has its own permissions block.

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection findings by moving ${{ }} expressions from run: blocks into env: blocks across all affected files:

1. actions/instrument/deploy/action.yml: Moved all inputs.*, github.*, and steps.*.outputs.* expressions to env: blocks. The find-self step now sanitizes path values with tr -d '\n\r' before writing to GITHUB_OUTPUT. The Enable auto-merge step uses PR_NUMBER env var.

2. .github/workflows/autobackport.yml: Moved ${{ matrix.ref }}, ${{ github.sha }}, ${{ github.event_path }} to env vars. The prepare_pr step now sanitizes commit_title, author_name, author_email with tr -d '\n\r' before writing to GITHUB_OUTPUT. The gh pr merge call uses PR_NUMBER env var.

3. .github/workflows/build.yml: Moved ${{ matrix.architecture }} to MATRIX_ARCHITECTURE env var.

4. .github/workflows/publish.yml: Rewrote to move all ${{ github.event_path }}, ${{ github.token }}, ${{ github.actor }}, ${{ matrix.ref }}, ${{ steps.version.outputs.version }} to env vars.

5. .github/workflows/refresh_demos.yml: Rewrote to move all ${{ github.token }}, ${{ matrix.demo_directory }}, ${{ github.event_name }}, ${{ github.event_path }} to env vars.

6. .github/workflows/renovate.yml: Fixed all three gh pr merge calls to use PR_NUMBER env var.

7. .github/workflows/recompile_agentic_workflows.yml: Fixed gh pr merge call to use PR_NUMBER env var.

8. .github/workflows/test_shell.yml: Moved ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }}, ${{ matrix.version }}, ${{ github.token }} to env vars in linux-shell, windows-shell, performance, upgrade, system-upgrade, install-manual-os, and dependency jobs.

9. .github/workflows/init_fork.yml: Moved ${{ secrets.ACTIONS_GITHUB_TOKEN }} to ACTIONS_GITHUB_TOKEN_VAL env var.

10. .github/workflows/test_package_repositories.yml: Moved ${{ github.repository_owner }}, ${{ github.event.repository.name }}, ${{ github.token }} to env vars.

The permissions finding was noted as already passing per the check rules (every job has its own permissions block).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings:
1. build.yml line 68: Replaced ${{ matrix.architecture }} in run block with $MATRIX_ARCHITECTURE (already defined in env block)
2. test_shell.yml line 113: Moved ${{ secrets.DOCKERHUB_USERNAME }} and ${{ secrets.DOCKERHUB_TOKEN }} to env block, use $DOCKERHUB_USERNAME/$DOCKERHUB_TOKEN in run block
3. test_github.yml deploy job - multiple steps fixed:
   - config step: Moved TEST_GITHUB_TOKEN, REPOSITORY_TEMPLATE, github.workflow, github.ref_name to env block; sanitized repository value with tr -d '\n\r' before writing to GITHUB_OUTPUT (fixes github-env-injection)
   - DELETE step: Moved TEST_GITHUB_TOKEN, CONFIG_USER, CONFIG_REPOSITORY to env block
   - Create repo step: Moved TEST_GITHUB_TOKEN, CONFIG_REPOSITORY, MATRIX_PRIVATE to env block
   - git config step: Moved CONFIG_USER to env block
   - printf/wget/yq step: Moved TEST_GITHUB_TOKEN, github.token, github.repository, github.sha, CONFIG_USER, CONFIG_REPOSITORY, github.repository/inputs.ref, matrix.secret_redaction_strategy to env block
   - case statement step: Moved matrix.secret_redaction_strategy to env block
   - Final cleanup DELETE step: Moved TEST_GITHUB_TOKEN, CONFIG_USER, CONFIG_REPOSITORY to env block

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed all 5 github-env-injection instances in hardened/action/actions/instrument/deploy/action.yml:
1. 'Determine repository' step: replaced xargs pipe with read+printf+tr sanitization before writing repository= to GITHUB_OUTPUT.
2. 'Determine version' step: replaced xargs pipe with read+printf+tr sanitization before writing version= to GITHUB_OUTPUT.
3. 'Find workflow-level observability' step: replaced bare echo path="$workflow_file" with printf+tr sanitization before writing path= to GITHUB_OUTPUT.
4. 'Find check-suite-level observability' step: same fix as #3.
5. 'Find repository-level observability' step: same fix as #3.
All fixes use printf '%s' ... | tr -d '\n\r' to strip newlines/carriage-returns before writing to $GITHUB_OUTPUT, preventing newline injection attacks.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed 6 script injection occurrences across 2 files:

1. hardened/action/.github/workflows/build.yml (3 fixes):
   - build-python-site-packages job: moved `${{ matrix.version }}` out of the `find -L` command into an `env: MATRIX_VERSION` block, referencing it as `"$MATRIX_VERSION"` in the shell.
   - build-java-agents job: moved `${{ steps.determine-minimum-version.outputs.version }}` out of the `echo` command into an `env: JAVA_VERSION` block, referencing it as `"$JAVA_VERSION"`.
   - build-deb job: moved `${{ needs.list-python-versions.outputs.versions }}` out of the `printf` command into an `env: PYTHON_VERSIONS` block, referencing it as `"$PYTHON_VERSIONS"`.

2. hardened/action/.github/workflows/test_github.yml (3 fixes):
   - Two `workflow_run` steps (in different jobs): moved `${{ secrets.GITHUB_TOKEN }}` into `env: GITHUB_TOKEN_VALUE` blocks, referencing it as `$GITHUB_TOKEN_VALUE` in the curl Authorization headers.
   - One `check_suite` step: moved `${{ secrets.GITHUB_TOKEN }}` into an `env: GITHUB_TOKEN_VALUE` block, referencing it as `$GITHUB_TOKEN_VALUE` in the curl Authorization headers.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in .github/workflows/test_github.yml by moving ${{ }} expressions out of run: shell commands and into env: blocks. Changes made: (1) job-io-1: moved steps.my-step.outputs.foo to MY_STEP_FOO env var; (2) job-io-2: moved needs.job-io-1.outputs.foo to JOB_IO_1_FOO env var; (3) workflow-smoke: moved steps.workflow_run.outputs.id and steps.workflow_run.outputs.run_attempt to WORKFLOW_RUN_ID and WORKFLOW_RUN_ATTEMPT env vars; (4) workflow job: same workflow_run outputs moved to env vars; (5) workflow job gh_artifact_download step: moved secrets.GITHUB_TOKEN, workflow_run.outputs.id, and workflow_run.outputs.run_attempt to INPUT_GITHUB_TOKEN, WORKFLOW_RUN_ID, and WORKFLOW_RUN_ATTEMPT env vars; (6) checksuite-smoke: moved steps.check_suite.outputs.id to CHECK_SUITE_ID env var. All shell commands now reference plain $VAR_NAME environment variables instead of ${{ }} expressions.

