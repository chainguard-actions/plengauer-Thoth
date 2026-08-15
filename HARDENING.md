<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.60.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.60.1** was hardened automatically. 7 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple run: blocks in actions/instrument/deploy-local/action.yml directly interpolate ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions inside shell commands. Examples include: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`, `ls "${{ inputs.workflows_directory }}"/*.yaml`, `if [ "${{ inputs.action_version }}" = same ]`, `echo '${{ inputs.action_version }}'`, `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`, and many more. Any ${{ }} expression inside a run: block is interpolated by the YAML template engine before the shell sees it, enabling script injection.

Locations:

- `actions/instrument/deploy-local/action.yml:68`
- `actions/instrument/deploy-local/action.yml:80`
- `actions/instrument/deploy-local/action.yml:95`
- `actions/instrument/deploy-local/action.yml:110`
- `actions/instrument/deploy-local/action.yml:120`

### script-injection (severity: high)

Rule (a): actions/instrument/deploy/action.yml has ${{ inputs.commit_message }} directly interpolated in a run: block: `git commit -m "${{ inputs.commit_message }}"`. Also `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` in the Enable auto-merge step. These expressions are expanded by the YAML template engine before the shell processes them, enabling command injection via crafted input values.

Locations:

- `actions/instrument/deploy/action.yml:155`
- `actions/instrument/deploy/action.yml:175`

### script-injection (severity: high)

Rule (a): .github/workflows/init_fork.yml has ${{ secrets.ACTIONS_GITHUB_TOKEN }} directly interpolated in run: blocks: `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]` and `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...`. Even though secrets are not attacker-controlled, any ${{ }} expression in a run: block goes through YAML template substitution before the shell, which is a script-injection pattern.

Locations:

- `.github/workflows/init_fork.yml:17`
- `.github/workflows/init_fork.yml:19`

### script-injection (severity: high)

Rule (a): .github/workflows/test_package_repositories.yml has ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} directly interpolated in run: blocks: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"` and `--header "Authorization: Bearer ${{ github.token }}"`. These github context values are interpolated by the YAML template engine before the shell, enabling script injection.

Locations:

- `.github/workflows/test_package_repositories.yml:18`
- `.github/workflows/test_package_repositories.yml:26`

### script-injection (severity: high)

Rule (a): .github/workflows/test_shell.yml has ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }}, and ${{ matrix.version }} directly interpolated in run: blocks: `bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'` and `sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb`. Matrix values flow through YAML template substitution before the shell.

Locations:

- `.github/workflows/test_shell.yml:55`
- `.github/workflows/test_shell.yml:80`
- `.github/workflows/test_shell.yml:82`

### script-injection (severity: high)

Rule (a): Multiple workflow files have ${{ steps.open-pr.outputs.pull-request-number }} directly interpolated in run: blocks: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. Step outputs are workflow-controllable and go through YAML template substitution before the shell. Affected files: autobackport.yml, renovate.yml (multiple occurrences), recompile_agentic_workflows.yml.

Locations:

- `.github/workflows/autobackport.yml:80`
- `.github/workflows/renovate.yml:95`
- `.github/workflows/renovate.yml:175`
- `.github/workflows/renovate.yml:200`
- `.github/workflows/recompile_agentic_workflows.yml:55`

### github-env-injection (severity: high)

actions/instrument/deploy-local/action.yml writes values derived from ${{ inputs.* }} and ${{ github.* }} directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Examples: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`, `echo version='{}' >> "$GITHUB_OUTPUT"` (where the version comes from ${{ inputs.action_version }}), and `echo "comment=# $comment" >> "$GITHUB_OUTPUT"` (where comment is derived from inputs). An attacker-controlled newline in any of these values could inject arbitrary environment variables or outputs.

Locations:

- `actions/instrument/deploy-local/action.yml:69`
- `actions/instrument/deploy-local/action.yml:71`
- `actions/instrument/deploy-local/action.yml:76`
- `actions/instrument/deploy-local/action.yml:110`
- `actions/instrument/deploy-local/action.yml:121`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 7 files:

1. actions/instrument/deploy-local/action.yml: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions from run: blocks into env: blocks across all 16 steps. Added tr -d '\n\r' sanitization for GITHUB_OUTPUT writes in the Find self step.

2. actions/instrument/deploy/action.yml: Moved inputs.commit_message to env block (COMMIT_MESSAGE) for git commit step; moved steps.open-pr.outputs.pull-request-number to env block (PR_NUMBER) for gh pr merge step.

3. .github/workflows/init_fork.yml: Moved secrets.ACTIONS_GITHUB_TOKEN to env blocks for both run: steps.

4. .github/workflows/test_package_repositories.yml: Moved github.repository_owner and github.event.repository.name to env block (REPO_OWNER/REPO_NAME); moved github.token to env block (GITHUB_TOKEN).

5. .github/workflows/test_shell.yml: Moved matrix.image/update/shell to env block for linux-shell job; moved matrix.shell to env block for windows-shell job; moved matrix.version and github.token to env blocks for upgrade and performance jobs.

6. .github/workflows/autobackport.yml: Moved steps.open-pr.outputs.pull-request-number to env block (PR_NUMBER) for gh pr merge step.

7. .github/workflows/renovate.yml: Moved steps.open-pr.outputs.pull-request-number to env block (PR_NUMBER) for all 3 gh pr merge steps.

8. .github/workflows/recompile_agentic_workflows.yml: Moved steps.open-pr.outputs.pull-request-number to env block (PR_NUMBER) for gh pr merge step.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 6 security findings:

1. publish.yml: Moved github.event_path, github.token, github.actor, and steps.version.outputs.version out of run: blocks into env: blocks.

2. autobackport.yml: Moved github.sha, github.event_path, and matrix.ref out of run: blocks into env: blocks.

3. build.yml: Moved matrix.architecture into env: block (MATRIX_ARCHITECTURE) for the build-http job, and matrix.version into env: block (MATRIX_VERSION) for the build-python-site-packages job's run step that used it in a path.

4. refresh_demos.yml: Moved github.token, github.event_name, github.event_path, and matrix.demo_directory out of run: blocks into env: blocks.

5. deploy-local/action.yml (Determine repository step, line 103): Replaced xargs pipe writing to GITHUB_OUTPUT with a while loop that sanitizes via printf '%s' | tr -d '\n\r' before writing.

6. deploy-local/action.yml (Determine version step, line 116): Replaced xargs pipe writing to GITHUB_OUTPUT with a while loop that sanitizes via printf '%s' | tr -d '\n\r' before writing.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed three unsanitized $GITHUB_OUTPUT writes in actions/instrument/deploy-local/action.yml. In each of the three steps ('Find workflow-level observability', 'Find check-suite-level observability', 'Find repository-level observability'), replaced `echo path="$workflow_file" >> "$GITHUB_OUTPUT"` with `safe=$(printf '%s' "$workflow_file" | tr -d '\n\r') && echo path="$safe" >> "$GITHUB_OUTPUT"`. This matches the sanitization pattern already used in other steps of the same file.

### Iteration 4

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four findings across three workflow files:

1. build.yml: Moved `${{ steps.determine-minimum-version.outputs.version }}` to an env var `JAVA_VERSION` to prevent script injection.

2. test_github.yml deploy job: Moved all ${{ }} expressions (secrets.TEST_GITHUB_TOKEN, steps.config.outputs.user/repository, matrix.private, github.token, github.repository, github.sha, inputs.ref, matrix.secret_redaction_strategy, env.REPOSITORY_TEMPLATE, github.workflow, github.ref_name) into env: blocks and referenced them as shell variables. Also sanitized the repository output with `tr -d '\n\r'` to prevent github-env-injection.

3. agentics-maintenance.yml: Added `printf '%s' ... | tr -d '\n\r'` sanitization for both the `operation` output (line 247) and `run_url` output (line 363) before writing to $GITHUB_OUTPUT.

### Iteration 5

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe shell pattern in hardened/action/actions/instrument/shared/install.sh at line 26. Replaced `wget -O - https://raw.githubusercontent.com/plengauer/Thoth/main/INSTALL.sh | sh -e` with a safe two-step approach: (1) create a temp file with mktemp, (2) download the script to the temp file with wget, (3) execute it with `sh -e "$INSTALL_SCRIPT"`, and (4) clean up with `rm -f`. The `-e` shell option is preserved. No `--` separator was present in the original pipe form, so none needed to be dropped.

