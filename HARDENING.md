<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.58.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.58.1** was hardened automatically. 13 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: steps in actions/instrument/deploy/action.yml directly interpolate ${{ ... }} expressions inside shell commands, violating rule (a). Examples include: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`, `elif [ -r "${{ github.workflow }}" ]`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`, `ls "${{ inputs.workflows_directory }}"/*.yaml`, `if [ -n "${{ inputs.action_repository }}" ]`, `version=${{ steps.determine-instrumentation-version.outputs.version }}`, and `run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. These allow an attacker-controlled value to be interpreted as shell code.

Locations:

- `actions/instrument/deploy/action.yml:88`
- `actions/instrument/deploy/action.yml:90`
- `actions/instrument/deploy/action.yml:92`
- `actions/instrument/deploy/action.yml:95`
- `actions/instrument/deploy/action.yml:100`
- `actions/instrument/deploy/action.yml:556`

### script-injection (severity: high)

run: blocks in .github/workflows/autobackport.yml directly interpolate ${{ github.sha }}, ${{ github.event_path }}, and ${{ matrix.ref }} inside shell commands (rule (a)). Example: `git describe --tags --abbrev=0 ${{ github.sha }}`, `cat '${{ github.event_path }}'`, `git format-patch -1 "${{ matrix.ref }}"`, `git log -1 --pretty=%s "${{ matrix.ref }}"`, and `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/autobackport.yml:28`
- `.github/workflows/autobackport.yml:29`
- `.github/workflows/autobackport.yml:74`
- `.github/workflows/autobackport.yml:85`
- `.github/workflows/autobackport.yml:91`
- `.github/workflows/autobackport.yml:107`

### script-injection (severity: high)

run: block in .github/workflows/recompile_agentic_workflows.yml directly interpolates ${{ steps.open-pr.outputs.pull-request-number }} inside a shell command (rule (a)): `run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. The steps.*.outputs.* context is workflow-controllable.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:52`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/renovate.yml directly interpolate ${{ steps.open-pr.outputs.pull-request-number }} inside shell commands (rule (a)): `run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` appears in three separate jobs (renovate-package-dependency-python, renovate-test-images, renovate-license).

Locations:

- `.github/workflows/renovate.yml:86`
- `.github/workflows/renovate.yml:183`
- `.github/workflows/renovate.yml:222`

### script-injection (severity: high)

run: blocks in .github/workflows/test_shell.yml directly interpolate ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }}, and ${{ matrix.version }} inside shell commands (rule (a)). Examples: `bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"`, `sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb`, and `performance.${{ matrix.version }}.coldstart.list`.

Locations:

- `.github/workflows/test_shell.yml:279`
- `.github/workflows/test_shell.yml:282`
- `.github/workflows/test_shell.yml:349`
- `.github/workflows/test_shell.yml:352`

### script-injection (severity: high)

run: blocks in .github/workflows/build.yml directly interpolate ${{ matrix.architecture }} inside shell commands (rule (a)): `--platform linux/${{ matrix.architecture }} docker.io/"$(echo ${{ matrix.architecture }} | tr -d /)"` and the result is written to $GITHUB_OUTPUT.

Locations:

- `.github/workflows/build.yml:62`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/test_github.yml directly interpolate ${{ github.workflow }}, ${{ github.ref_name }}, ${{ env.REPOSITORY_TEMPLATE }}, ${{ secrets.TEST_GITHUB_TOKEN }}, and ${{ github.token }} inside shell commands (rule (a)). Example: `echo repository="$(echo "${{ env.REPOSITORY_TEMPLATE }}"-"${{ github.workflow }}"-"${{ github.ref_name }}" | tr / - | cut -c -100)" >> "$GITHUB_OUTPUT"`.

Locations:

- `.github/workflows/test_github.yml:855`
- `.github/workflows/test_github.yml:856`

### script-injection (severity: high)

run: blocks in .github/workflows/publish.yml directly interpolate ${{ github.event_path }}, ${{ steps.version.outputs.version }}, ${{ github.token }}, ${{ github.actor }}, and ${{ matrix.ref }} inside shell commands (rule (a)). Example: `echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin` and `version="${{ steps.version.outputs.version }}"`.

Locations:

- `.github/workflows/publish.yml:30`
- `.github/workflows/publish.yml:62`
- `.github/workflows/publish.yml:63`
- `.github/workflows/publish.yml:89`

### script-injection (severity: high)

run: blocks in .github/workflows/refresh_demos.yml directly interpolate ${{ github.token }} and ${{ matrix.demo_directory }} inside shell commands (rule (a)). Examples: `wget --header "Authorization: Bearer ${{ github.token }}"`, `export GITHUB_TOKEN=${{ github.token }}`, and `cd demos/${{ matrix.demo_directory }}`.

Locations:

- `.github/workflows/refresh_demos.yml:30`
- `.github/workflows/refresh_demos.yml:31`
- `.github/workflows/refresh_demos.yml:44`
- `.github/workflows/refresh_demos.yml:45`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy/action.yml writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). An attacker-controlled input containing newlines could inject arbitrary environment variables.

Locations:

- `actions/instrument/deploy/action.yml:88`
- `actions/instrument/deploy/action.yml:90`

### github-env-injection (severity: high)

The 'dynamic' step in .github/workflows/autobackport.yml writes values derived from ${{ github.sha }} and ${{ matrix.ref }} directly to $GITHUB_OUTPUT without sanitization. Examples: `echo "has_tags=true" >> "$GITHUB_OUTPUT"` after using ${{ github.sha }} in a git command, and `echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"`. A crafted commit title could inject additional output variables.

Locations:

- `.github/workflows/autobackport.yml:28`
- `.github/workflows/autobackport.yml:85`
- `.github/workflows/autobackport.yml:86`
- `.github/workflows/autobackport.yml:87`

### github-env-injection (severity: high)

A run: step in .github/workflows/build.yml writes a value derived from ${{ matrix.architecture }} directly to $GITHUB_OUTPUT without sanitization: `echo 'architecture={}' >> "$GITHUB_OUTPUT"` where the value is derived from `$(echo ${{ matrix.architecture }} | tr -d /)`. A matrix value containing newlines could inject additional output variables.

Locations:

- `.github/workflows/build.yml:62`

### github-env-injection (severity: high)

A run: step in .github/workflows/test_github.yml writes values derived from ${{ github.workflow }}, ${{ github.ref_name }}, and ${{ env.REPOSITORY_TEMPLATE }} directly to $GITHUB_OUTPUT without sanitization: `echo repository="$(echo "${{ env.REPOSITORY_TEMPLATE }}"-"${{ github.workflow }}"-"${{ github.ref_name }}" | tr / - | cut -c -100)" >> "$GITHUB_OUTPUT"`. A crafted workflow name or ref name containing newlines could inject additional output variables.

Locations:

- `.github/workflows/test_github.yml:855`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 9 files:

1. actions/instrument/deploy/action.yml:
   - 'Find self' step: moved __repository_level_instrumentation_file_name_override, github.workflow, workflows_directory to env vars; sanitized outputs with tr -d '\n\r' before writing to GITHUB_OUTPUT
   - 'Determine repository' step: moved inputs.action_repository to env var
   - 3 startup optimization steps: moved steps.determine-instrumentation-version.outputs.version to env var INSTRUMENTATION_VERSION
   - 'Enable auto-merge' step: moved steps.open-pr.outputs.pull-request-number to env var PR_NUMBER

2. .github/workflows/autobackport.yml:
   - 'dynamic' step: moved github.sha and github.event_path to env vars
   - 'prepare_branch' step: moved matrix.ref to env var MATRIX_REF
   - format-patch step: moved matrix.ref to env var MATRIX_REF
   - 'prepare_pr' step: moved matrix.ref to env var; sanitized commit_title, author_name, author_email with tr -d '\n\r' before writing to GITHUB_OUTPUT
   - 'gh pr merge' step: moved steps.open-pr.outputs.pull-request-number to env var PR_NUMBER

3. .github/workflows/recompile_agentic_workflows.yml:
   - 'gh pr merge' step: moved steps.open-pr.outputs.pull-request-number to env var PR_NUMBER

4. .github/workflows/renovate.yml:
   - All 3 'gh pr merge' steps: moved steps.open-pr.outputs.pull-request-number to env var PR_NUMBER

5. .github/workflows/test_shell.yml:
   - linux-shell job: moved matrix.image, matrix.update, matrix.shell to env vars
   - performance job: moved matrix.version to env var MATRIX_VERSION in all run steps; also moved github.token to env var

6. .github/workflows/build.yml:
   - build step: moved matrix.architecture to env var MATRIX_ARCHITECTURE; computed arch_nodash from env var; sanitized architecture output with tr -d '\n\r' before writing to GITHUB_OUTPUT

7. .github/workflows/test_github.yml:
   - config step: moved secrets.TEST_GITHUB_TOKEN, env.REPOSITORY_TEMPLATE, github.workflow, github.ref_name to env vars; sanitized repository output with tr -d '\n\r' before writing to GITHUB_OUTPUT

8. .github/workflows/publish.yml:
   - setup/dynamic step: moved github.event_path and github.token to env vars
   - publish step (docker login): moved steps.version.outputs.version, github.token, github.actor, matrix.ref to env vars
   - publish step (git tag): moved steps.version.outputs.version to env var

9. .github/workflows/refresh_demos.yml:
   - First run step: moved github.token to env var GITHUB_TOKEN_VAL
   - demo step: moved github.token and matrix.demo_directory to env vars; saved token before GITHUB_ unset loop
   - Second run step: moved matrix.demo_directory to env var
   - Third run step (git add): moved matrix.demo_directory to env var

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection findings across 6 files by moving every ${{ }} expression out of run: shell blocks into the step's env: block, then referencing them as plain environment variables ($VAR_NAME) in the shell scripts.

1. actions/instrument/deploy/action.yml: Added env: blocks to 'Determine version', 'Canonicalize', 'Find workflow/check-suite/repository-level observability', 'Deploy workflow/check-suite/repository-level observability', 'Deploy *-level startup optimization', 'Update workflow-level observability triggers', 'Deploy Copilot Setup', 'Deploy job-level instrumentation', 'Configure job-level instrumentation secret redaction', 'Modify permissions', 'Restore blank lines', and 'Push' steps.

2. .github/workflows/build.yml: Fixed build-java-agents step (steps.determine-minimum-version.outputs.version) and build-deb step (needs.list-python-versions.outputs.versions).

3. .github/workflows/init_fork.yml: Fixed both run steps that used secrets.ACTIONS_GITHUB_TOKEN directly.

4. .github/workflows/test_github.yml: Fixed the deploy job's curl DELETE, curl POST, git config, gh secret set/yq commands, and case statement.

5. .github/workflows/test_shell.yml: Fixed upgrade job (github.token, matrix.version), list-images job (DOCKERHUB credentials), system-upgrade job (matrix.image), dependency job (matrix.image), install-manual-os job (matrix.image), and windows-shell job (matrix.shell).

6. .github/workflows/test_package_repositories.yml: Fixed the apt sources list echo and the curl command.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 6 script-injection locations in .github/workflows/test_github.yml by moving ${{ steps.*.outputs.* }} expressions into step env: blocks and referencing them as plain shell variables. Fixed 2 github-env-injection locations in actions/instrument/deploy/action.yml by replacing unsanitized `xargs -I '{}' echo key='{}' >> $GITHUB_OUTPUT` patterns with sanitized while loops using `printf '%s' "$_val" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.

