<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.58.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.58.1** was hardened automatically. 12 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Direct expression interpolation in run: blocks in actions/instrument/deploy/action.yml. Multiple steps interpolate ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} directly into shell commands. Examples: 'if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]' (Find self step), 'if [ "${{ inputs.action_version }}" = same ]' (Determine version step), 'sed -i 's~'"$repository"'~${{ steps.determine-repository.outputs.repository }}~g' (Canonicalize step), 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}' (Enable auto-merge step). This violates rule (a): any ${{ ... }} expression directly inside a run: script is a script-injection finding.

Locations:

- `actions/instrument/deploy/action.yml:57`
- `actions/instrument/deploy/action.yml:68`
- `actions/instrument/deploy/action.yml:75`
- `actions/instrument/deploy/action.yml:82`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/autobackport.yml. Examples: 'git describe --tags --abbrev=0 ${{ github.sha }}' (setup step), 'cat '${{ github.event_path }}'' (setup step), 'git describe --tags --abbrev=0 ${{ matrix.ref }}' (prepare_branch step), 'git format-patch -1 "${{ matrix.ref }}"' (apply patch step), 'git log -1 --pretty=%s "${{ matrix.ref }}"' (prepare_pr step), 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}' (final step). Violates rule (a).

Locations:

- `.github/workflows/autobackport.yml:29`
- `.github/workflows/autobackport.yml:30`
- `.github/workflows/autobackport.yml:60`
- `.github/workflows/autobackport.yml:68`
- `.github/workflows/autobackport.yml:76`
- `.github/workflows/autobackport.yml:91`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/build.yml. The build-http job interpolates ${{ matrix.architecture }} directly into shell commands: 'debian_architecture="$(echo ${{ matrix.architecture }} | cut -d / -f 1 | sed 's/le$/el/g')"' and 'sudo docker pull --platform linux/${{ matrix.architecture }} docker.io/"$(echo ${{ matrix.architecture }} | tr -d /)"'. Violates rule (a).

Locations:

- `.github/workflows/build.yml:47`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/init_fork.yml. The init job interpolates ${{ secrets.ACTIONS_GITHUB_TOKEN }} directly into shell commands: 'curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}"'. Violates rule (a).

Locations:

- `.github/workflows/init_fork.yml:22`
- `.github/workflows/init_fork.yml:25`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/publish.yml. Examples: 'version="${{ steps.version.outputs.version }}"', 'echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin', 'for tag_simple in ... "${{ matrix.ref }}"'. Also in the final run block: 'version="${{ steps.version.outputs.version }}"'. Violates rule (a).

Locations:

- `.github/workflows/publish.yml:79`
- `.github/workflows/publish.yml:83`
- `.github/workflows/publish.yml:91`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/refresh_demos.yml. Examples: 'curl --header "Authorization: Bearer ${{ github.token }}"', 'export GITHUB_TOKEN=${{ github.token }}', 'cd demos/${{ matrix.demo_directory }}', 'sed -i s/${{ github.token }}/***/g otlp.json'. Violates rule (a).

Locations:

- `.github/workflows/refresh_demos.yml:42`
- `.github/workflows/refresh_demos.yml:57`
- `.github/workflows/refresh_demos.yml:58`
- `.github/workflows/refresh_demos.yml:97`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/test_package_repositories.yml. The test-apt job interpolates ${{ github.repository_owner }} and ${{ github.event.repository.name }} directly into a shell command: 'echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"'. Also interpolates ${{ github.token }} directly. Violates rule (a).

Locations:

- `.github/workflows/test_package_repositories.yml:18`
- `.github/workflows/test_package_repositories.yml:25`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/test_shell.yml. Examples: 'bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'' (linux-shell job), 'sudo docker run --rm --network=host -i --entrypoint=/bin/sh ${{ matrix.image }} -e <<EOF' (install-manual-os job), 'sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb' (performance job), 'wget --header "Authorization: Bearer ${{ github.token }}"' (upgrade job). Violates rule (a).

Locations:

- `.github/workflows/test_shell.yml:131`
- `.github/workflows/test_shell.yml:118`
- `.github/workflows/test_shell.yml:175`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/recompile_agentic_workflows.yml. The recompile job interpolates ${{ steps.open-pr.outputs.pull-request-number }} directly into a shell command: 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}'. Violates rule (a).

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:50`

### script-injection (severity: high)

Direct expression interpolation in run: blocks in .github/workflows/renovate.yml. Multiple jobs interpolate ${{ steps.open-pr.outputs.pull-request-number }} directly into shell commands: 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}'. Violates rule (a).

Locations:

- `.github/workflows/renovate.yml:55`
- `.github/workflows/renovate.yml:130`
- `.github/workflows/renovate.yml:148`

### github-env-injection (severity: high)

In actions/instrument/deploy/action.yml, the 'Find self' step writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without sanitization: 'echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"' and 'echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"'. The 'Determine version' step also writes inputs.action_version-derived values to $GITHUB_OUTPUT without sanitization. No printf '%s' ... | tr -d '\n\r' sanitization is applied.

Locations:

- `actions/instrument/deploy/action.yml:58`
- `actions/instrument/deploy/action.yml:60`
- `actions/instrument/deploy/action.yml:75`

### github-env-injection (severity: high)

In .github/workflows/autobackport.yml, the 'prepare_pr' step writes values derived from ${{ matrix.ref }} to $GITHUB_OUTPUT without sanitization: 'echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"', 'echo author_name="$(git log -1 --pretty=%an "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"', 'echo author_email="$(git log -1 --pretty=%ae "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"'. No sanitization is applied before writing.

Locations:

- `.github/workflows/autobackport.yml:76`
- `.github/workflows/autobackport.yml:77`
- `.github/workflows/autobackport.yml:78`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings by moving ${{ }} expressions from run: blocks into step env: blocks and referencing them as plain environment variables in shell scripts.

Files modified:
1. actions/instrument/deploy/action.yml: Fixed Find self step (inputs.__repository_level_instrumentation_file_name_override, github.workflow), Determine repository step (steps.find-self.outputs.path, inputs.action_repository), Determine version step (inputs.action_version, steps.find-self.outputs.path, steps.determine-repository.outputs.repository), Canonicalize step (steps.determine-repository.outputs.repository), Enable auto-merge step (steps.open-pr.outputs.pull-request-number). Added tr -d newlines sanitization for GITHUB_OUTPUT writes.
2. .github/workflows/autobackport.yml: Fixed setup dynamic step (github.sha, github.event_path), prepare_branch step (matrix.ref), apply patch step (matrix.ref), prepare_pr step (matrix.ref with GITHUB_OUTPUT sanitization), final merge step (steps.open-pr.outputs.pull-request-number).
3. .github/workflows/build.yml: Fixed build-http build step (matrix.architecture).
4. .github/workflows/init_fork.yml: Fixed curl step (secrets.ACTIONS_GITHUB_TOKEN).
5. .github/workflows/publish.yml: Fixed setup dynamic step (github.event_path, github.token), publish docker step (steps.version.outputs.version, github.token, github.actor, matrix.ref), final tag step (steps.version.outputs.version).
6. .github/workflows/refresh_demos.yml: Fixed download step (github.token), demo run step (github.token, matrix.demo_directory, sed token replacement), second demo step (matrix.demo_directory), git diff step (matrix.demo_directory).
7. .github/workflows/test_package_repositories.yml: Fixed apt sources step (github.repository_owner, github.event.repository.name), curl step (github.token).
8. .github/workflows/test_shell.yml: Fixed system-upgrade step (matrix.image), dependency step (matrix.image), install-manual-os step (matrix.image), linux-shell step (matrix.image, matrix.update, matrix.shell), windows-shell step (matrix.shell), upgrade step (github.token, matrix.version), performance step (github.token, matrix.version).
9. .github/workflows/recompile_agentic_workflows.yml: Fixed gh pr merge step (steps.open-pr.outputs.pull-request-number).
10. .github/workflows/renovate.yml: Fixed three gh pr merge steps (steps.open-pr.outputs.pull-request-number) in renovate-package-dependency-python, renovate-test-images, and renovate-license jobs.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection findings by moving ${{ }} expressions from run: blocks to env: blocks in: (1) actions/instrument/deploy/action.yml - moved inputs.workflows_directory, inputs.workflow_level_instrumentation_workflow_name, steps.determine-repository.outputs.repository, steps.determine-instrumentation-version.outputs.version, inputs.workflow_level_instrumentation_file_name, steps.find-self.outputs.path, steps.find-workflow-level-instrumentation.outputs.path, github.repository_owner, inputs.github_token, inputs.workflow_level_instrumentation_exclude, steps.find-check-suite-level-instrumentation.outputs.path, inputs.check_suite_instrumentation_file_name, steps.find-repository-level-instrumentation.outputs.path, inputs.repository_level_instrumentation_file_name, github.repository, inputs.job_level_instrumentation_exclude, inputs.workflow_level_instrumentation_file_name, inputs.job_level_instrumentation_secret_redaction_strategy, inputs.commit_message to env: blocks; (2) .github/workflows/init_fork.yml - moved secrets.ACTIONS_GITHUB_TOKEN to env: block; (3) .github/workflows/build.yml - moved matrix.version, steps.determine-minimum-version.outputs.version, needs.list-python-versions.outputs.versions to env: blocks; (4) .github/workflows/test_github.yml - moved secrets.TEST_GITHUB_TOKEN, env.REPOSITORY_TEMPLATE, github.workflow, github.ref_name, github.repository, github.sha, matrix.secret_redaction_strategy, steps.config.outputs.user, steps.config.outputs.repository, inputs.ref to env: blocks; (5) Fixed github-env-injection by sanitizing the repository output with tr -d '\n\r' before writing to GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection issues in test_github.yml (8 locations) and test_shell.yml (1 location) by moving ${{ ... }} expressions out of run: shell blocks into env: blocks. Fixed github-env-injection in actions/instrument/deploy/action.yml by adding `| tr -d '\n\r'` before writing the repository value to GITHUB_OUTPUT to strip attacker-controlled newlines.

