<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.61.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.61.2** was hardened automatically. 11 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks in composite action steps. The 'Find self' step interpolates ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} directly into shell commands. The 'Determine repository' step interpolates ${{ inputs.action_repository }}, ${{ steps.find-self.outputs.path }}, and ${{ github.action_repository }}. The 'Determine version' step interpolates ${{ inputs.action_version }} and ${{ steps.find-self.outputs.path }}. Many subsequent steps similarly interpolate ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} directly into shell commands throughout the file. These values are attacker-controllable and allow shell command injection.

Locations:

- `actions/instrument/deploy-local/action.yml:74`
- `actions/instrument/deploy-local/action.yml:76`
- `actions/instrument/deploy-local/action.yml:80`
- `actions/instrument/deploy-local/action.yml:90`
- `actions/instrument/deploy-local/action.yml:96`
- `actions/instrument/deploy-local/action.yml:100`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks in composite action steps. The 'Push' step interpolates ${{ inputs.commit_message }} directly into a git commit command: `git commit -m "${{ inputs.commit_message }}"`; The 'Enable auto-merge' step interpolates ${{ steps.open-pr.outputs.pull-request-number }} directly into a gh CLI command: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. These values are attacker-controllable.

Locations:

- `actions/instrument/deploy/action.yml:148`
- `actions/instrument/deploy/action.yml:163`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. The build-http job interpolates ${{ matrix.architecture }} directly into shell commands: `echo ${{ matrix.architecture }} | cut -d / -f 1` and `docker pull --platform linux/${{ matrix.architecture }}`. The build-python job interpolates ${{ matrix.version }} into a find path. These matrix values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/build.yml:57`
- `.github/workflows/build.yml:59`
- `.github/workflows/build.yml:261`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. Multiple steps interpolate ${{ matrix.* }} and ${{ steps.config.outputs.* }} directly into shell commands, e.g.: `curl ... -d '{"name":"${{ steps.config.outputs.repository }}","private":${{ matrix.private }},...}'` and `case "${{ matrix.secret_redaction_strategy }}" in`. These values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/test_github.yml:763`
- `.github/workflows/test_github.yml:780`
- `.github/workflows/test_github.yml:840`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. The setup job interpolates ${{ github.sha }} into a git command: `git describe --tags --abbrev=0 ${{ github.sha }}`, and ${{ github.event_path }} into a shell command: `cat '${{ github.event_path }}' | jq '.commits[].id' -r`. These values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/autobackport.yml:35`
- `.github/workflows/autobackport.yml:36`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. The setup job interpolates ${{ github.event_path }} into a shell command: `cat '${{ github.event_path }}' | jq ...`. The publish job interpolates ${{ steps.version.outputs.version }}, ${{ github.token }}, and ${{ github.actor }} directly into shell commands including `echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin`. These values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/publish.yml:36`
- `.github/workflows/publish.yml:88`
- `.github/workflows/publish.yml:91`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. A step interpolates ${{ github.repository_owner }} and ${{ github.event.repository.name }} directly into a shell command: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main" | sudo tee /etc/apt/sources.list.d/otel.list`. These values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/test_package_repositories.yml:20`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. A step interpolates ${{ github.token }} directly into wget/curl commands: `wget --header "Authorization: Bearer ${{ github.token }}"`. Another step interpolates ${{ matrix.demo_directory }} into a shell command: `cd demos/${{ matrix.demo_directory }}`. These values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/refresh_demos.yml:47`
- `.github/workflows/refresh_demos.yml:183`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. Multiple steps interpolate ${{ github.token }} and ${{ matrix.version }} directly into shell commands, e.g.: `wget --header "Authorization: Bearer ${{ github.token }}" ... releases/tags/${{ matrix.version }}`. These values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/test_shell.yml:57`
- `.github/workflows/test_shell.yml:58`

### script-injection (severity: high)

Rule (a): Direct ${{ }} expression interpolation inside run: blocks. Steps interpolate ${{ secrets.ACTIONS_GITHUB_TOKEN }} directly into shell commands: `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]` and `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...`. These values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/init_fork.yml:21`
- `.github/workflows/init_fork.yml:23`

### github-env-injection (severity: high)

The 'Find self' step in the composite action writes untrusted input values directly to $GITHUB_OUTPUT without sanitization. Specifically: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. An attacker-controlled newline in these values could inject arbitrary environment variables or outputs. The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `actions/instrument/deploy-local/action.yml:74`
- `actions/instrument/deploy-local/action.yml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 10 files:

1. actions/instrument/deploy-local/action.yml: Moved ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, ${{ inputs.workflows_directory }} (Find self step), ${{ inputs.action_repository }}, ${{ steps.find-self.outputs.path }}, ${{ github.action_repository }} (Determine repository step), ${{ inputs.action_version }} (Determine version and Determine version comment steps) to env: blocks. Added tr -d '\n\r' sanitization for GITHUB_OUTPUT writes to fix github-env-injection.

2. actions/instrument/deploy/action.yml: Moved ${{ inputs.commit_message }} (Push step) and ${{ steps.open-pr.outputs.pull-request-number }} (Enable auto-merge step) to env: blocks.

3. .github/workflows/build.yml: Moved ${{ matrix.architecture }} (build-http step) and ${{ matrix.version }} (build-python-site-packages step) to env: blocks.

4. .github/workflows/test_github.yml: Moved ${{ steps.config.outputs.repository }}, ${{ matrix.private }} (curl repo creation), ${{ github.token }}, ${{ github.repository }}, ${{ github.sha }}, ${{ matrix.secret_redaction_strategy }}, ${{ inputs.ref }} (wget/yq step), and ${{ matrix.secret_redaction_strategy }} (case statement) to env: blocks.

5. .github/workflows/autobackport.yml: Moved ${{ github.sha }} and ${{ github.event_path }} to env: block.

6. .github/workflows/publish.yml: Moved ${{ github.event_path }}, ${{ github.token }} (setup job), ${{ steps.version.outputs.version }}, ${{ github.token }}, ${{ github.actor }} (docker login step), and ${{ steps.version.outputs.version }} (is_latest and final tag steps) to env: blocks.

7. .github/workflows/test_package_repositories.yml: Moved ${{ github.repository_owner }} and ${{ github.event.repository.name }} to env: block.

8. .github/workflows/refresh_demos.yml: Moved ${{ github.token }} (wget step) and ${{ matrix.demo_directory }} (cd commands) to env: blocks.

9. .github/workflows/test_shell.yml: Moved ${{ github.token }} and ${{ matrix.version }} to env: block.

10. .github/workflows/init_fork.yml: Moved ${{ secrets.ACTIONS_GITHUB_TOKEN }} to env: blocks in both affected steps.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 9 findings across 7 files:

1. actions/instrument/deploy-local/action.yml (script-injection + github-env-injection): Rewrote the entire file to move all ${{ inputs.* }} and ${{ steps.*.outputs.* }} expressions from run: blocks into env: blocks. Also fixed Determine repository, Determine version, and Determine version comment steps to sanitize values with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

2. .github/workflows/test_shell.yml (script-injection): Moved ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }} to env: block as MATRIX_IMAGE, MATRIX_UPDATE, MATRIX_SHELL.

3. .github/workflows/test_github.yml (script-injection): Moved ${{ steps.my-step.outputs.foo }} to env: as STEP_FOO and ${{ needs.job-io-1.outputs.foo }} to env: as JOB_IO_1_FOO.

4. .github/workflows/autobackport.yml (script-injection): Moved ${{ steps.open-pr.outputs.pull-request-number }} to env: as PR_NUMBER.

5. .github/workflows/renovate.yml (script-injection): Fixed all 3 occurrences of ${{ steps.open-pr.outputs.pull-request-number }} by moving to env: as PR_NUMBER.

6. .github/workflows/recompile_agentic_workflows.yml (script-injection): Moved ${{ steps.open-pr.outputs.pull-request-number }} to env: as PR_NUMBER.

7. .github/workflows/test_package_repositories.yml (script-injection): Moved ${{ github.token }} to env: as GH_TOKEN.

8. .github/workflows/agentics-maintenance.yml (github-env-injection): Added sanitization before writing GH_AW_OPERATION and GH_AW_RUN_URL to $GITHUB_OUTPUT in both Record outputs steps.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 6 findings across 3 files:

1. test_shell.yml - system-upgrade job: Added MATRIX_IMAGE env var, replaced ${{ matrix.image }} in docker run command.
2. test_shell.yml - dependency job: Added MATRIX_IMAGE to existing env block, replaced ${{ matrix.image }} in docker run command.
3. test_shell.yml - install-manual-os job: Added MATRIX_IMAGE env var, replaced ${{ matrix.image }} in docker run command.
4. test_shell.yml - windows-shell job: Added MATRIX_SHELL env var, replaced ${{ matrix.shell }} in conditionals and bash -c command.
5. test_shell.yml - performance job: Added GITHUB_TOKEN_VALUE and MATRIX_VERSION env vars to each run step, replaced all ${{ github.token }} and ${{ matrix.version }} expressions.
6. autobackport.yml - prepare_branch step: Added MATRIX_REF env var, replaced ${{ matrix.ref }} in git describe command.
7. autobackport.yml - backport step: Added MATRIX_REF env var, replaced ${{ matrix.ref }} in git format-patch and git cherry-pick commands.
8. autobackport.yml - prepare_pr step: Added MATRIX_REF env var, replaced ${{ matrix.ref }} in git log commands, AND fixed github-env-injection by piping git log output through tr -d '\n\r' and using printf to write to GITHUB_OUTPUT.
9. refresh_demos.yml - demo step: Replaced ${{ github.token }} with $GITHUB_TOKEN_VALUE in sed command (env var already existed).
10. refresh_demos.yml - second step: Added MATRIX_DEMO_DIRECTORY env var, replaced ${{ matrix.demo_directory }} with "$MATRIX_DEMO_DIRECTORY" in cd command.

### Iteration 4

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings:

1. build.yml: Moved `${{ needs.list-python-versions.outputs.versions }}` from run: shell string to env: block as PYTHON_VERSIONS.

2. test_github.yml (multiple locations):
   - workflow-smoke: Moved secrets.GITHUB_TOKEN to env block for workflow_run step; moved step outputs to env block for assertion step.
   - workflow job: Same pattern for workflow_run step, assertion step, and gh_artifact_download step.
   - checksuite-smoke: Moved secrets.GITHUB_TOKEN to env block; moved check_suite output to env block for assertion.
   - config step: Moved TEST_GITHUB_TOKEN, env.REPOSITORY_TEMPLATE, github.workflow, github.ref_name to env block; sanitized repository name with tr -d '\n\r' before writing to GITHUB_OUTPUT (fixes github-env-injection).
   - DELETE curl steps (2): Moved TEST_GITHUB_TOKEN and config outputs to env block.
   - git config step: Moved steps.config.outputs.user to env block.

3. test_shell.yml: Moved secrets.DOCKERHUB_USERNAME and secrets.DOCKERHUB_TOKEN from run: shell string to env: block.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings:
1. hardened/action/.github/workflows/publish.yml: Moved `${{ matrix.ref }}` from the shell script's `for` loop into the step's `env:` block as `MATRIX_REF`, and updated the shell script to reference `"$MATRIX_REF"` instead.
2. hardened/action/.github/workflows/build.yml: Moved `${{ steps.determine-minimum-version.outputs.version }}` from the `echo` command into the step's `env:` block as `MINIMUM_VERSION`, and updated the shell script to reference `"$MINIMUM_VERSION"` instead.

