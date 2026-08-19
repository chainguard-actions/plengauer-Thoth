<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.55.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.55.0** was hardened automatically. 14 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in actions/instrument/deploy/action.yml directly interpolate ${{ }} expressions. Sub-rule (a) violations include: 'if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]' (Find self step), 'elif [ -r "${{ github.workflow }}" ]' (Find self step), 'echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"' (Find self step), 'echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"' (Find self step), 'if [ -n "${{ inputs.action_repository }}" ]' (Determine repository step), 'if [ "${{ inputs.action_version }}" = same ]' (Determine version step), 'echo '${{ inputs.action_version }}'' (Determine version step), 'sed -i 's~'"$repository"'~${{ steps.determine-repository.outputs.repository }}~g'' (Canonicalize step), 'echo 'name: ${{ inputs.workflow_level_instrumentation_workflow_name }}'' (Deploy workflow-level step), 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}' (Enable auto-merge step), and many more throughout the file. All these allow an attacker who controls inputs to inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy/action.yml:100`
- `actions/instrument/deploy/action.yml:102`
- `actions/instrument/deploy/action.yml:104`
- `actions/instrument/deploy/action.yml:106`
- `actions/instrument/deploy/action.yml:120`
- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:145`
- `actions/instrument/deploy/action.yml:600`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/autobackport.yml directly interpolate ${{ }} expressions. Sub-rule (a) violations: 'git describe --tags --abbrev=0 ${{ github.sha }}' (dynamic step), 'cat '${{ github.event_path }}'' (dynamic step), 'git describe --tags --abbrev=0 ${{ matrix.ref }}' (prepare_branch step), 'git format-patch -1 "${{ matrix.ref }}"' (apply step), 'git cherry-pick -n "${{ matrix.ref }}"' (apply step), 'git log -1 --pretty=%s "${{ matrix.ref }}"' (prepare_pr step), 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}' (merge step). These allow injection via github.sha, github.event_path, matrix.ref, and steps outputs.

Locations:

- `.github/workflows/autobackport.yml:35`
- `.github/workflows/autobackport.yml:37`
- `.github/workflows/autobackport.yml:60`
- `.github/workflows/autobackport.yml:72`
- `.github/workflows/autobackport.yml:75`
- `.github/workflows/autobackport.yml:83`
- `.github/workflows/autobackport.yml:100`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/publish.yml directly interpolate ${{ }} expressions. Sub-rule (a) violations: 'cat '${{ github.event_path }}'' (dynamic step), '${{ github.token }}' used in curl (dynamic step), 'echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin' (publish step), 'version="${{ steps.version.outputs.version }}"' (publish step), '${{ matrix.ref }}' used in docker tag loop (publish step). These allow injection via github.event_path, github.token, github.actor, steps outputs, and matrix values.

Locations:

- `.github/workflows/publish.yml:35`
- `.github/workflows/publish.yml:37`
- `.github/workflows/publish.yml:80`
- `.github/workflows/publish.yml:82`
- `.github/workflows/publish.yml:90`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/refresh_demos.yml directly interpolate ${{ }} expressions. Sub-rule (a) violations: 'curl --header "Authorization: Bearer ${{ github.token }}"' (generate step), 'wget --header "Authorization: Bearer ${{ github.token }}"' (generate step), 'export GITHUB_TOKEN=${{ github.token }}' (demo step), 'cd demos/${{ matrix.demo_directory }}' (demo step), 'sed -i s/${{ github.token }}/***/g otlp.json' (demo step). These allow injection via github.token and matrix.demo_directory.

Locations:

- `.github/workflows/refresh_demos.yml:55`
- `.github/workflows/refresh_demos.yml:65`
- `.github/workflows/refresh_demos.yml:70`
- `.github/workflows/refresh_demos.yml:75`
- `.github/workflows/refresh_demos.yml:80`

### script-injection (severity: high)

A run: block in .github/workflows/build.yml directly interpolates ${{ matrix.architecture }} multiple times. Sub-rule (a) violations: 'debian_architecture="$(echo ${{ matrix.architecture }} | cut -d / -f 1 | sed 's/le$/el/g')"', 'sudo docker pull --platform linux/${{ matrix.architecture }} docker.io/"$(echo ${{ matrix.architecture }} | tr -d /)"', etc. Although matrix values are hardcoded in this workflow, ${{ matrix.* }} is a workflow-controllable context and must not be interpolated directly in run: blocks.

Locations:

- `.github/workflows/build.yml:36`

### script-injection (severity: high)

Multiple run: blocks in .github/workflows/test_shell.yml directly interpolate ${{ }} expressions. Sub-rule (a) violations: 'bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'' (containerized test step), 'sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb' (install step), '2>> performance.${{ matrix.version }}.coldstart.list' (performance step). These allow injection via matrix values.

Locations:

- `.github/workflows/test_shell.yml:350`
- `.github/workflows/test_shell.yml:430`
- `.github/workflows/test_shell.yml:435`

### script-injection (severity: high)

run: blocks in .github/workflows/renovate.yml and .github/workflows/recompile_agentic_workflows.yml directly interpolate ${{ steps.open-pr.outputs.pull-request-number }} in 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}'. Sub-rule (a) violation: steps.*.outputs.* is a workflow-controllable context that must not be interpolated directly in run: blocks.

Locations:

- `.github/workflows/renovate.yml:75`
- `.github/workflows/renovate.yml:115`
- `.github/workflows/renovate.yml:145`
- `.github/workflows/recompile_agentic_workflows.yml:55`

### script-injection (severity: high)

run: blocks in .github/workflows/init_fork.yml directly interpolate ${{ secrets.ACTIONS_GITHUB_TOKEN }} in shell commands: '[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]' and 'curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}"'. Sub-rule (a) violation: any ${{ }} expression directly in a run: block is a script injection risk regardless of context.

Locations:

- `.github/workflows/init_fork.yml:18`
- `.github/workflows/init_fork.yml:20`

### script-injection (severity: high)

run: blocks in .github/workflows/test_package_repositories.yml directly interpolate ${{ }} expressions: 'echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"' and '--header "Authorization: Bearer ${{ github.token }}"'. Sub-rule (a) violation: github.repository_owner, github.event.repository.name, and github.token are interpolated directly in shell commands.

Locations:

- `.github/workflows/test_package_repositories.yml:17`
- `.github/workflows/test_package_repositories.yml:24`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy/action.yml writes untrusted input values directly to $GITHUB_OUTPUT without sanitization (no printf '%s' ... | tr -d '\n\r' step): 'echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"' and 'echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"'. An attacker controlling these inputs could inject newlines to set arbitrary environment variables.

Locations:

- `actions/instrument/deploy/action.yml:102`
- `actions/instrument/deploy/action.yml:104`

### github-env-injection (severity: high)

The 'Determine repository' step in actions/instrument/deploy/action.yml writes ${{ inputs.action_repository }} to $GITHUB_OUTPUT via xargs without sanitization: 'echo '${{ inputs.action_repository }}' | ... | xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"'. Similarly, the 'Determine version' step writes ${{ inputs.action_version }} to $GITHUB_OUTPUT via xargs without sanitization. An attacker controlling these inputs could inject newlines.

Locations:

- `actions/instrument/deploy/action.yml:120`
- `actions/instrument/deploy/action.yml:130`

### github-env-injection (severity: high)

The 'prepare_pr' step in .github/workflows/autobackport.yml writes values derived from ${{ matrix.ref }} (a workflow-controllable context) to $GITHUB_OUTPUT without sanitization: 'echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"', 'echo author_name=... >> "$GITHUB_OUTPUT"', 'echo author_email=... >> "$GITHUB_OUTPUT"'. Git commit messages can contain newlines, enabling injection of arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/autobackport.yml:83`
- `.github/workflows/autobackport.yml:84`
- `.github/workflows/autobackport.yml:85`

### unpinned-uses (severity: high)

The workflow .github/workflows/experiment.yml uses 'actions/checkout@v6' which is a mutable tag reference, not a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/workflows/experiment.yml:26`

### unpinned-uses (severity: high)

The workflow .github/workflows/test_github.yml uses 'actions/setup-dotnet@v5' which is a mutable tag reference, not a full 40-character commit SHA. This is vulnerable to supply-chain attacks if the tag is moved.

Locations:

- `.github/workflows/test_github.yml:115`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all script injection findings by moving ${{ }} expressions from run: blocks into step env: blocks and referencing them as plain environment variables. Fixed github-env-injection by adding tr -d '\n\r' sanitization before writing to GITHUB_OUTPUT. Pinned unpinned action references (actions/checkout@v6 → SHA, actions/setup-dotnet@v5 → SHA). Files modified: actions/instrument/deploy/action.yml, .github/workflows/autobackport.yml, .github/workflows/publish.yml, .github/workflows/refresh_demos.yml, .github/workflows/build.yml, .github/workflows/test_shell.yml, .github/workflows/renovate.yml, .github/workflows/recompile_agentic_workflows.yml, .github/workflows/init_fork.yml, .github/workflows/test_package_repositories.yml, .github/workflows/experiment.yml, .github/workflows/test_github.yml.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. build.yml script-injection (3 locations): Moved matrix.version, steps.determine-minimum-version.outputs.version, and needs.list-python-versions.outputs.versions from direct ${{ }} interpolation in run: blocks to env: variables.

2. test_github.yml script-injection (12 locations): Moved all ${{ }} expressions from run: blocks to env: variables including secrets.GITHUB_TOKEN (→ GITHUB_TOKEN_VAL), steps.my-step.outputs.foo (→ MY_STEP_FOO), needs.job-io-1.outputs.foo (→ JOB_IO_1_FOO), steps.workflow_run.outputs.id/run_attempt (→ WORKFLOW_RUN_ID/WORKFLOW_RUN_ATTEMPT), steps.check_suite.outputs.id (→ CHECK_SUITE_ID), secrets.TEST_GITHUB_TOKEN (→ TEST_GITHUB_TOKEN), github.token/repository/sha (→ GITHUB_TOKEN_VAL/GITHUB_REPOSITORY_VAL/GITHUB_SHA_VAL), matrix.secret_redaction_strategy (→ MATRIX_SECRET_REDACTION_STRATEGY), steps.config.outputs.user/repository (→ CONFIG_USER/CONFIG_REPOSITORY), matrix.private (→ MATRIX_PRIVATE), inputs.ref (→ INPUTS_REF).

3. refresh_demos.yml script-injection (line 80): Moved matrix.demo_directory from cd demos/${{ matrix.demo_directory }} to env var DEMO_DIRECTORY.

4. actions/instrument/deploy/action.yml github-env-injection (lines 115, 130): Replaced xargs-based GITHUB_OUTPUT writes with newline-stripping pattern using printf '%s' | tr -d '\n\r' for both repository and version outputs.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the `deploy` job's `config` step in `.github/workflows/test_github.yml`. The `GITHUB_WORKFLOW_VAL` (from `${{ github.workflow }}`) and `GITHUB_REF_NAME_VAL` (from `${{ github.ref_name }}`) variables were being used directly in an `echo repository=...` line that writes to `$GITHUB_OUTPUT` without stripping newlines. The fix introduces two sanitization steps using `printf '%s' "$VAR" | tr -d '\n\r'` to create `safe_workflow` and `safe_ref_name` variables, which are then used in the `echo repository=...` line instead of the raw env vars. This prevents a newline in either github.* value from injecting additional key=value pairs into GITHUB_OUTPUT.

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in `.github/workflows/test_shell.yml` at the `list-images` job's docker login step. Moved `${{ secrets.DOCKERHUB_USERNAME }}` and `${{ secrets.DOCKERHUB_TOKEN }}` from the `run:` shell string into an `env:` block as `DOCKERHUB_USERNAME` and `DOCKERHUB_TOKEN` respectively. The shell command now uses `"$DOCKERHUB_USERNAME"` and `"$DOCKERHUB_TOKEN"` (double-quoted environment variable references) instead of directly interpolating the GitHub Actions expressions.

