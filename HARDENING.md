<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.61.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.61.0** was hardened automatically. 14 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in actions/instrument/deploy-local/action.yml directly interpolate ${{ }} expressions inside shell commands (sub-rule a). Examples include: 'Find self' step uses ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly in shell; 'Determine repository' uses ${{ inputs.action_repository }} and ${{ steps.find-self.outputs.path }}; 'Determine version' uses ${{ inputs.action_version }}; 'Determine version comment' uses ${{ inputs.action_version }}; 'Canonicalize' uses ${{ inputs.workflows_directory }} and ${{ steps.determine-repository.outputs.repository }}; 'Find workflow/check-suite/repository-level observability' steps use ${{ inputs.workflows_directory }}; 'Deploy workflow-level observability' uses ${{ inputs.workflow_level_instrumentation_workflow_name }}, ${{ inputs.workflows_directory }}, ${{ inputs.workflow_level_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}; 'Deploy workflow-level startup optimization' uses ${{ github.repository_owner }}, ${{ inputs.github_token }}, ${{ steps.determine-instrumentation-version.outputs.version }}; 'Update workflow-level observability triggers' uses ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.github_token }}, ${{ github.repository }}; 'Deploy check suite-level instrumentation' and startup optimization use similar patterns; 'Deploy repository-level instrumentation' and startup optimization use similar patterns; 'Deploy Copilot Setup' uses ${{ inputs.workflows_directory }}; 'Deploy job-level instrumentation' and 'Configure job-level instrumentation secret redaction' use ${{ inputs.job_level_instrumentation_exclude }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}; 'Annotate instrumentation action versions' uses ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ steps.determine-repository.outputs.repository }}. All of these allow an attacker-controlled value to be interpreted as shell code.

Locations:

- `actions/instrument/deploy-local/action.yml:72`
- `actions/instrument/deploy-local/action.yml:88`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:120`
- `actions/instrument/deploy-local/action.yml:133`
- `actions/instrument/deploy-local/action.yml:143`
- `actions/instrument/deploy-local/action.yml:152`
- `actions/instrument/deploy-local/action.yml:161`
- `actions/instrument/deploy-local/action.yml:170`
- `actions/instrument/deploy-local/action.yml:190`
- `actions/instrument/deploy-local/action.yml:210`
- `actions/instrument/deploy-local/action.yml:240`
- `actions/instrument/deploy-local/action.yml:260`
- `actions/instrument/deploy-local/action.yml:280`
- `actions/instrument/deploy-local/action.yml:300`
- `actions/instrument/deploy-local/action.yml:320`
- `actions/instrument/deploy-local/action.yml:335`
- `actions/instrument/deploy-local/action.yml:360`
- `actions/instrument/deploy-local/action.yml:390`
- `actions/instrument/deploy-local/action.yml:410`
- `actions/instrument/deploy-local/action.yml:430`

### script-injection (severity: high)

The 'Push' step in actions/instrument/deploy/action.yml directly interpolates ${{ inputs.commit_message }} inside a git commit -m shell command (sub-rule a): `git commit -m "${{ inputs.commit_message }}"`  — an attacker-controlled commit message could inject shell metacharacters. The 'Enable auto-merge' step also directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} in a run: block: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:150`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/autobackport.yml directly interpolate ${{ }} expressions (sub-rule a): the 'dynamic' step uses `cat '${{ github.event_path }}'` and `git describe --tags --abbrev=0 ${{ github.sha }}`; the 'prepare_branch' step uses `${{ matrix.ref }}` in git commands; a run step uses `git format-patch -1 "${{ matrix.ref }}"` and `git cherry-pick -n "${{ matrix.ref }}"`; the 'prepare_pr' step uses `git log -1 --pretty=%s "${{ matrix.ref }}"`; and a final run step uses `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/autobackport.yml:30`
- `.github/workflows/autobackport.yml:70`
- `.github/workflows/autobackport.yml:75`
- `.github/workflows/autobackport.yml:100`
- `.github/workflows/autobackport.yml:130`

### script-injection (severity: high)

Three run: blocks in .github/workflows/renovate.yml directly interpolate ${{ steps.open-pr.outputs.pull-request-number }} in shell commands (sub-rule a): `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. This appears in three separate jobs.

Locations:

- `.github/workflows/renovate.yml:110`
- `.github/workflows/renovate.yml:240`
- `.github/workflows/renovate.yml:280`

### script-injection (severity: high)

A run: block in .github/workflows/recompile_agentic_workflows.yml directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} in a shell command (sub-rule a): `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:50`

### script-injection (severity: high)

The 'build' step in .github/workflows/build.yml directly interpolates ${{ matrix.architecture }} multiple times inside a run: shell command (sub-rule a): `--platform linux/${{ matrix.architecture }} docker.io/"$(echo ${{ matrix.architecture }} | tr -d /)"`. The matrix value flows through YAML template substitution before the shell parses it.

Locations:

- `.github/workflows/build.yml:50`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/refresh_demos.yml directly interpolate ${{ matrix.demo_directory }} inside shell commands (sub-rule a): `cd demos/${{ matrix.demo_directory }}`. This appears in at least three steps.

Locations:

- `.github/workflows/refresh_demos.yml:50`
- `.github/workflows/refresh_demos.yml:85`
- `.github/workflows/refresh_demos.yml:230`

### script-injection (severity: high)

A run: block in .github/workflows/test_shell.yml directly interpolates ${{ matrix.image }}, ${{ matrix.update }}, and ${{ matrix.shell }} inside a shell command (sub-rule a): `bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'`. Additional run: blocks also interpolate ${{ matrix.version }} directly.

Locations:

- `.github/workflows/test_shell.yml:330`
- `.github/workflows/test_shell.yml:380`
- `.github/workflows/test_shell.yml:385`

### script-injection (severity: high)

Two run: blocks in .github/workflows/init_fork.yml directly interpolate ${{ secrets.ACTIONS_GITHUB_TOKEN }} inside curl shell commands (sub-rule a): `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...`. Secrets interpolated directly into run: blocks are expanded before the shell runs and can contain newlines or metacharacters.

Locations:

- `.github/workflows/init_fork.yml:16`
- `.github/workflows/init_fork.yml:21`

### script-injection (severity: high)

A run: block in .github/workflows/test_package_repositories.yml directly interpolates ${{ github.token }} inside a curl shell command (sub-rule a): `--header "Authorization: Bearer ${{ github.token }}"`.

Locations:

- `.github/workflows/test_package_repositories.yml:22`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy-local/action.yml writes untrusted input values directly to $GITHUB_OUTPUT without sanitization: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. An attacker-controlled value containing newlines could inject additional key=value pairs into the output file.

Locations:

- `actions/instrument/deploy-local/action.yml:75`

### github-env-injection (severity: high)

The 'Determine repository' step in actions/instrument/deploy-local/action.yml writes the value of ${{ inputs.action_repository }} to $GITHUB_OUTPUT without sanitization: `echo '${{ inputs.action_repository }}'` is piped through `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"`. An attacker-controlled value containing newlines could inject additional key=value pairs.

Locations:

- `actions/instrument/deploy-local/action.yml:88`

### github-env-injection (severity: high)

The 'Determine version' step in actions/instrument/deploy-local/action.yml writes the value of ${{ inputs.action_version }} to $GITHUB_OUTPUT without sanitization: `echo '${{ inputs.action_version }}'` is piped through `xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"`. An attacker-controlled value containing newlines could inject additional key=value pairs.

Locations:

- `actions/instrument/deploy-local/action.yml:107`

### github-env-injection (severity: high)

The 'Determine version comment' step in actions/instrument/deploy-local/action.yml writes the value of ${{ inputs.action_version }} (via the $comment shell variable) to $GITHUB_OUTPUT without sanitization: `comment='${{ inputs.action_version }}'` followed by `echo "comment=# $comment" >> "$GITHUB_OUTPUT"`. An attacker-controlled value containing newlines could inject additional key=value pairs.

Locations:

- `actions/instrument/deploy-local/action.yml:120`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings across 10 files:

1. actions/instrument/deploy-local/action.yml: Added env: blocks to all 20+ run: steps that used ${{ }} expressions directly in shell. Moved inputs.__repository_level_instrumentation_file_name_override, github.workflow, inputs.workflows_directory, inputs.action_repository, steps.find-self.outputs.path, inputs.action_version, steps.determine-repository.outputs.repository, steps.determine-instrumentation-version.outputs.version, inputs.workflow_level_instrumentation_workflow_name, inputs.workflow_level_instrumentation_file_name, github.repository_owner, inputs.github_token, inputs.workflow_level_instrumentation_exclude, inputs.job_level_instrumentation_exclude, inputs.job_level_instrumentation_secret_redaction_strategy, steps.determine-instrumentation-version-comment.outputs.comment, and others to env vars. Also sanitized GITHUB_OUTPUT writes with tr -d '\n\r' for github-env-injection fixes.

2. actions/instrument/deploy/action.yml: Moved inputs.commit_message to COMMIT_MESSAGE env var in Push step; moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in Enable auto-merge step.

3. .github/workflows/autobackport.yml: Moved github.sha and github.event_path to env vars in dynamic step; moved matrix.ref to MATRIX_REF env var in prepare_branch, format-patch/cherry-pick, and prepare_pr steps; moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var.

4. .github/workflows/renovate.yml: Moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var in all three jobs (renovate-package-dependency-python, renovate-test-images, renovate-license).

5. .github/workflows/recompile_agentic_workflows.yml: Moved steps.open-pr.outputs.pull-request-number to PR_NUMBER env var.

6. .github/workflows/build.yml: Moved matrix.architecture to MATRIX_ARCHITECTURE env var in build step; pre-computed arch_nodash to avoid repeated subshell expansion.

7. .github/workflows/refresh_demos.yml: Moved github.token to GITHUB_TOKEN_VAL and matrix.demo_directory to DEMO_DIRECTORY env vars in three steps.

8. .github/workflows/test_shell.yml: Moved matrix.image/update/shell to env vars in linux-shell step; moved matrix.shell to MATRIX_SHELL in windows-shell step; moved github.token and matrix.version to env vars in upgrade and performance steps.

9. .github/workflows/init_fork.yml: Moved secrets.ACTIONS_GITHUB_TOKEN to ACTIONS_GITHUB_TOKEN_VAL env var in both run steps.

10. .github/workflows/test_package_repositories.yml: Moved github.token to GITHUB_TOKEN_VAL env var in the curl step.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 workflow files:

1. build.yml: Moved ${{ matrix.version }}, ${{ steps.determine-minimum-version.outputs.version }}, and ${{ needs.list-python-versions.outputs.versions }} from run: blocks to env: blocks.

2. publish.yml: Moved ${{ github.event_path }}, ${{ github.token }}, ${{ github.actor }}, ${{ steps.version.outputs.version }}, and ${{ matrix.ref }} from run: blocks to env: blocks.

3. test_github.yml: Fixed all flagged run: blocks by moving ${{ secrets.GITHUB_TOKEN }}, ${{ secrets.TEST_GITHUB_TOKEN }}, ${{ steps.*.outputs.* }}, ${{ needs.*.outputs.* }}, ${{ github.workflow }}, ${{ github.ref_name }}, ${{ github.repository }}, ${{ github.sha }}, ${{ github.token }}, ${{ inputs.ref }}, and ${{ matrix.secret_redaction_strategy }} to env: blocks. Also fixed the github-env-injection finding by using printf '%s' ... | tr -d '\n\r' to sanitize the repository value before writing to GITHUB_OUTPUT.

4. test_shell.yml: Moved ${{ secrets.DOCKERHUB_USERNAME }}, ${{ secrets.DOCKERHUB_TOKEN }}, and ${{ matrix.image }} from run: blocks to env: blocks.

5. test_package_repositories.yml: Moved ${{ github.repository_owner }} and ${{ github.event.repository.name }} from run: block to env: block.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/.github/workflows/agentics-maintenance.yml:
1. In the 'run_operation' job's 'Record outputs' step (line ~375): Changed `echo "operation=$GH_AW_OPERATION" >> "$GITHUB_OUTPUT"` to use `safe=$(printf '%s' "$GH_AW_OPERATION" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
2. In the 'apply_safe_outputs' job's 'Record outputs' step (line ~525): Changed `echo "run_url=$GH_AW_RUN_URL" >> "$GITHUB_OUTPUT"` to use `safe=$(printf '%s' "$GH_AW_RUN_URL" | tr -d '\n\r')` before writing to GITHUB_OUTPUT.
Both fixes prevent newline injection attacks where a caller-controlled value containing '\n' could inject arbitrary key-value pairs into $GITHUB_OUTPUT.

