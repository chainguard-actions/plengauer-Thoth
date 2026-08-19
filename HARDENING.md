<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.55.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.55.1** was hardened automatically. 12 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` steps in actions/instrument/deploy/action.yml directly interpolate GitHub Actions expressions inside shell commands (sub-rule a). Examples include:
- 'Find self' step: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `echo path="${{ github.workflow }}"` and `ls "${{ inputs.workflows_directory }}"`
- 'Determine repository' step: `cat "${{ steps.find-self.outputs.path }}"` and `if [ -n "${{ inputs.action_repository }}" ]`
- 'Determine version' step: `if [ "${{ inputs.action_version }}" = same ]`
- 'Canonicalize' step: `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`
- Many subsequent deploy steps with `${{ inputs.* }}`, `${{ steps.*.outputs.* }}`, and `${{ github.* }}` directly in shell commands.
Any of these expressions can contain shell metacharacters injected by a caller of this composite action.

Locations:

- `actions/instrument/deploy/action.yml:91`
- `actions/instrument/deploy/action.yml:95`
- `actions/instrument/deploy/action.yml:97`
- `actions/instrument/deploy/action.yml:110`
- `actions/instrument/deploy/action.yml:115`
- `actions/instrument/deploy/action.yml:125`

### script-injection (severity: high)

The 'build-http' job in build.yml directly interpolates `${{ matrix.architecture }}` inside a multiline `run:` shell script (sub-rule a). Example: `debian_architecture="$(echo ${{ matrix.architecture }} | cut -d / -f 1 ...)"` and `sudo docker pull --platform linux/${{ matrix.architecture }} ...`. The 'build-python-site-packages' job similarly uses `${{ matrix.version }}` directly in a run block: `find -L .../python${{ matrix.version }}/site-packages ...`.

Locations:

- `.github/workflows/build.yml:51`
- `.github/workflows/build.yml:355`

### script-injection (severity: high)

Multiple `run:` steps in test_shell.yml directly interpolate `${{ matrix.* }}` and `${{ github.token }}` expressions inside shell commands (sub-rule a). Examples:
- 'upgrade' job: `curl ... --header "Authorization: Bearer ${{ github.token }}" ... ${{ matrix.version }}`
- 'system-upgrade' job: `sudo docker run ... --entrypoint=/bin/sh ${{ matrix.image }} -e <<EOF`
- 'dependency' job: `sudo -E docker run ... --entrypoint=/bin/sh ${{ matrix.image }} -e <<EOF`
- 'install-manual-os' job: `sudo docker run ... --entrypoint=/bin/sh ${{ matrix.image }} -e <<EOF`
- 'linux-shell' job: `bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"`

Locations:

- `.github/workflows/test_shell.yml:85`
- `.github/workflows/test_shell.yml:250`
- `.github/workflows/test_shell.yml:330`
- `.github/workflows/test_shell.yml:380`
- `.github/workflows/test_shell.yml:430`

### script-injection (severity: high)

The 'generate' job in refresh_demos.yml directly interpolates `${{ github.token }}` and `${{ matrix.demo_directory }}` inside `run:` shell commands (sub-rule a). Examples: `export GITHUB_TOKEN=${{ github.token }}`, `cd demos/${{ matrix.demo_directory }}`, and `sed -i s/${{ github.token }}/***/g otlp.json`.

Locations:

- `.github/workflows/refresh_demos.yml:60`
- `.github/workflows/refresh_demos.yml:62`
- `.github/workflows/refresh_demos.yml:100`

### script-injection (severity: high)

The 'backport' job in autobackport.yml directly interpolates `${{ matrix.ref }}` inside `run:` shell commands (sub-rule a). Examples: `current_tag=$(git describe --tags --abbrev=0 ${{ matrix.ref }})`, `git format-patch -1 "${{ matrix.ref }}" --stdout`, `git cherry-pick -n "${{ matrix.ref }}"`, and `git log -1 --pretty=%s "${{ matrix.ref }}"`.

Locations:

- `.github/workflows/autobackport.yml:75`
- `.github/workflows/autobackport.yml:90`
- `.github/workflows/autobackport.yml:100`

### script-injection (severity: high)

The 'publish' job in publish.yml directly interpolates `${{ steps.version.outputs.version }}`, `${{ github.token }}`, and `${{ github.actor }}` inside a `run:` shell command (sub-rule a). Example: `version="${{ steps.version.outputs.version }}"` and `echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin`.

Locations:

- `.github/workflows/publish.yml:95`

### script-injection (severity: high)

The 'recompile' job in recompile_agentic_workflows.yml and multiple jobs in renovate.yml directly interpolate `${{ steps.open-pr.outputs.pull-request-number }}` inside `run:` shell commands (sub-rule a). Example: `run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:55`
- `.github/workflows/renovate.yml:80`
- `.github/workflows/renovate.yml:130`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy/action.yml writes values derived from untrusted inputs directly to $GITHUB_OUTPUT without sanitization. Specifically: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write.

Locations:

- `actions/instrument/deploy/action.yml:95`
- `actions/instrument/deploy/action.yml:97`

### github-env-injection (severity: high)

The 'Determine version' step in actions/instrument/deploy/action.yml writes `${{ inputs.action_version }}` to $GITHUB_OUTPUT without sanitization: `echo '${{ inputs.action_version }}'` is piped via xargs to `echo version='{}' >> "$GITHUB_OUTPUT"`. No `tr -d '\n\r'` sanitization is applied.

Locations:

- `actions/instrument/deploy/action.yml:115`

### github-env-injection (severity: high)

The 'Determine repository' step in actions/instrument/deploy/action.yml writes a value derived from `${{ inputs.action_repository }}` (and `${{ github.action_repository }}` via env var) to $GITHUB_OUTPUT without sanitization: `fi | xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"`. No `tr -d '\n\r'` sanitization is applied.

Locations:

- `actions/instrument/deploy/action.yml:110`

### github-env-injection (severity: high)

The 'prepare_pr' step in autobackport.yml writes values derived from `${{ matrix.ref }}` (a workflow-controlled value) to $GITHUB_OUTPUT without sanitization. The git log output of the commit referenced by `${{ matrix.ref }}` is written directly: `echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"`. A malicious commit message could inject newlines to poison GITHUB_OUTPUT.

Locations:

- `.github/workflows/autobackport.yml:105`

### unpinned-uses (severity: high)

The 'experiment' job in experiment.yml uses `actions/checkout@v6` which is a mutable tag reference, not a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/workflows/experiment.yml:23`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all 12 findings across 9 files:

1. experiment.yml: Pinned actions/checkout@v6 to full SHA d23441a48e516b6c34aea4fa41551a30e30af803.

2. build.yml: Moved ${{ matrix.architecture }} and ${{ matrix.version }} into env: blocks (MATRIX_ARCHITECTURE, MATRIX_VERSION) for the build-http and build-python-site-packages jobs.

3. test_shell.yml: Moved ${{ github.token }}, ${{ matrix.version }}, ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }} into env: blocks for upgrade, system-upgrade, dependency, install-manual-os, and linux-shell jobs.

4. refresh_demos.yml: Moved ${{ github.token }} (as GITHUB_TOKEN_VAL) and ${{ matrix.demo_directory }} (as MATRIX_DEMO_DIRECTORY) into env: blocks. Fixed sed command to use env var instead of direct expression.

5. autobackport.yml: Moved ${{ matrix.ref }} into env: blocks (MATRIX_REF) for prepare_branch, format-patch/cherry-pick, and prepare_pr steps. Sanitized GITHUB_OUTPUT writes in prepare_pr using printf + tr -d '\n\r'. Fixed gh pr merge to use PR_NUMBER env var.

6. publish.yml: Moved ${{ steps.version.outputs.version }}, ${{ github.token }}, ${{ github.actor }} into env: blocks for the docker login/push step and the git tag step. Also fixed the setup job's dynamic step.

7. recompile_agentic_workflows.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} into env: block as PR_NUMBER.

8. renovate.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} into env: blocks as PR_NUMBER for both renovate-test-images and renovate-license jobs.

9. actions/instrument/deploy/action.yml: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions into env: blocks for Find self, Determine repository, Determine version, Canonicalize, Find *-level observability, Push, and Enable auto-merge steps. Applied tr -d '\n\r' sanitization for all GITHUB_OUTPUT writes in the affected steps.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection findings by moving ${{ }} expressions from run: shell blocks into step env: blocks. Changes made to:
1. actions/instrument/deploy/action.yml: Fixed 12 steps (Deploy workflow-level observability, Deploy workflow-level startup optimization, Update workflow-level observability triggers, Deploy check suite-level instrumentation, Deploy check suite-level startup optimization, Deploy repository-level instrumentation, Deploy repository-level startup optimization, Deploy Copilot Setup, Deploy job-level instrumentation, Configure job-level instrumentation secret redaction, Modify permissions for job-level instrumentations, Restore blank lines). All inputs.*, steps.*.outputs.*, and github.* context values moved to env: blocks and referenced as $VAR_NAME in shell scripts.
2. .github/workflows/test_shell.yml: Fixed windows-shell job (${{ matrix.shell }}) and performance job (${{ matrix.version }}, ${{ github.token }}) by adding env: blocks.
3. .github/workflows/init_fork.yml: Fixed ${{ secrets.ACTIONS_GITHUB_TOKEN }} in two run: steps by adding env: blocks.
4. .github/workflows/autobackport.yml: Fixed ${{ github.sha }} and ${{ github.event_path }} in the dynamic step by adding env: block.
5. .github/workflows/publish.yml: Fixed ${{ github.event_path }} in the dynamic step by adding it to the existing env: block.
6. .github/workflows/test_package_repositories.yml: Fixed ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} by adding env: blocks.
7. .github/workflows/refresh_demos.yml: Fixed ${{ matrix.demo_directory }} in the run: block by adding MATRIX_DEMO_DIRECTORY to the existing env: block.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection issues in .github/workflows/test_github.yml by moving ${{ }} expressions from run: blocks to env: blocks:

1. job-io-1: Moved ${{ steps.my-step.outputs.foo }} to env block as STEP_FOO
2. job-io-2: Moved ${{ needs.job-io-1.outputs.foo }} to env block as JOB_IO_1_FOO
3. workflow-smoke: Moved ${{ secrets.GITHUB_TOKEN }} to env block as GITHUB_TOKEN_VALUE in the curl step; moved ${{ steps.workflow_run.outputs.id }} and run_attempt to env block for the check step
4. workflow job: Same fixes as workflow-smoke, plus moved ${{ secrets.GITHUB_TOKEN }}, ${{ steps.workflow_run.outputs.id }}, and run_attempt to env block for the INPUT_GITHUB_TOKEN step
5. checksuite-smoke: Moved ${{ secrets.GITHUB_TOKEN }} to env block as GITHUB_TOKEN_VALUE; moved ${{ steps.check_suite.outputs.id }} to env block as CHECK_SUITE_ID
6. deploy job (config step): Moved ${{ secrets.TEST_GITHUB_TOKEN }}, ${{ env.REPOSITORY_TEMPLATE }}, ${{ github.workflow }}, ${{ github.ref_name }} to env block; fixed github-env-injection by sanitizing the repository value with tr -d '\n\r' before writing to GITHUB_OUTPUT
7. deploy job (DELETE curl steps): Moved ${{ secrets.TEST_GITHUB_TOKEN }}, ${{ steps.config.outputs.user }}, ${{ steps.config.outputs.repository }} to env blocks
8. deploy job (create repo curl step): Moved ${{ secrets.TEST_GITHUB_TOKEN }}, ${{ steps.config.outputs.repository }}, ${{ matrix.private }} to env block
9. deploy job (git config step): Moved ${{ steps.config.outputs.user }} to env block
10. deploy job (printf/wget/yq step): Moved ${{ secrets.TEST_GITHUB_TOKEN }}, ${{ github.token }}, ${{ github.repository }}, ${{ github.sha }}, ${{ inputs.ref }}, ${{ matrix.secret_redaction_strategy }}, ${{ steps.config.outputs.user }}, ${{ steps.config.outputs.repository }} to env block
11. deploy job (case statement): Moved ${{ matrix.secret_redaction_strategy }} to env block as MATRIX_SECRET_REDACTION_STRATEGY

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all 4 script-injection instances across 3 files:
1. hardened/action/.github/workflows/build.yml (line ~395): Moved `${{ steps.determine-minimum-version.outputs.version }}` into an env: block as JAVA_MIN_VERSION.
2. hardened/action/.github/workflows/build.yml (line ~530): Moved `${{ needs.list-python-versions.outputs.versions }}` into an env: block as PYTHON_VERSIONS.
3. hardened/action/.github/workflows/test_shell.yml (line ~107): Moved `${{ secrets.DOCKERHUB_USERNAME }}` and `${{ secrets.DOCKERHUB_TOKEN }}` into an env: block as DOCKERHUB_USERNAME and DOCKERHUB_TOKEN for the docker login command.
4. hardened/action/.github/workflows/publish.yml (line ~100): Moved `${{ matrix.ref }}` into an env: block as MATRIX_REF for use in the for-loop.

