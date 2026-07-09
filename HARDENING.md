<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.58.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.58.1** was hardened automatically. 11 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell command strings in the deploy composite action. Examples include: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `echo '${{ inputs.action_repository }}'`, `if [ "${{ inputs.action_version }}" = same ]`, `cat "${{ steps.find-self.outputs.path }}"`, `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`, `echo ${{ github.repository_owner }}`, `curl ... ${{ github.repository }}`, `git commit -m "${{ inputs.commit_message }}"`, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. These allow injection of arbitrary shell commands via attacker-controlled inputs.

Locations:

- `actions/instrument/deploy/action.yml:100`

### github-env-injection (severity: high)

Untrusted input values from inputs.* and github.* contexts are written directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Examples: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`, and `echo version='{}' >> "$GITHUB_OUTPUT"` where {} is substituted from inputs.action_version via xargs.

Locations:

- `actions/instrument/deploy/action.yml:100`

### script-injection (severity: high)

Rule (a): ${{ ... }} expressions are directly interpolated inside run: shell command strings. Examples: `[ -z "$(git describe --tags --abbrev=0 ${{ github.sha }})" ]`, `cat '${{ github.event_path }}'`, `current_tag=$(git describe --tags --abbrev=0 ${{ matrix.ref }})`, `git format-patch -1 "${{ matrix.ref }}"`, `git cherry-pick -n "${{ matrix.ref }}"`, `echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")"`, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/autobackport.yml:28`

### script-injection (severity: high)

Rule (a): ${{ ... }} expressions are directly interpolated inside run: shell command strings. Examples: `debian_architecture="$(echo ${{ matrix.architecture }} | cut -d / -f 1 | sed 's/le$/el/g')"`, `sudo docker pull --platform linux/${{ matrix.architecture }}`, `echo "${{ steps.determine-minimum-version.outputs.version }}" > version`, `printf '%s' "${{ needs.list-python-versions.outputs.versions }}" | jq -r '.[]'`.

Locations:

- `.github/workflows/build.yml:60`

### script-injection (severity: high)

Rule (a): ${{ ... }} expressions are directly interpolated inside run: shell command strings. Examples: `cat '${{ github.event_path }}'`, `curl -s --fail -H "Authorization: Bearer ${{ github.token }}"`, `echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin`, `version="${{ steps.version.outputs.version }}"`.

Locations:

- `.github/workflows/publish.yml:30`

### script-injection (severity: high)

Rule (a): ${{ ... }} expressions are directly interpolated inside run: shell command strings. Examples: `bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'`, `sudo docker run ... ${{ matrix.image }} -e <<EOF`, `[ ${{ matrix.shell }} = sh ] || sudo -E apt-get -y install ${{ matrix.shell }}`, `bash -c "cd tests && bash run_tests.sh ${{ matrix.shell }}"`, `curl ... --header "Authorization: Bearer ${{ github.token }}"`, `mv opentelemetry-shell_*_all.deb opentelemetry-shell_${{ matrix.version }}_all.deb`, `sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb`.

Locations:

- `.github/workflows/test_shell.yml:270`

### script-injection (severity: high)

Rule (a): ${{ steps.open-pr.outputs.pull-request-number }} is directly interpolated inside a run: shell command string: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/renovate.yml:120`

### script-injection (severity: high)

Rule (a): ${{ steps.open-pr.outputs.pull-request-number }} is directly interpolated inside a run: shell command string: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:55`

### script-injection (severity: high)

Rule (a): ${{ ... }} expressions are directly interpolated inside run: shell command strings. Examples: `curl --header "Authorization: Bearer ${{ github.token }}"`, `export GITHUB_TOKEN=${{ github.token }}`, `cd demos/${{ matrix.demo_directory }}`, `sed -i s/${{ github.token }}/***/g otlp.json`.

Locations:

- `.github/workflows/refresh_demos.yml:40`

### script-injection (severity: high)

Rule (a): ${{ secrets.ACTIONS_GITHUB_TOKEN }} is directly interpolated inside run: shell command strings: `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]`, `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ... | xargs -I '{}' curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}"`.

Locations:

- `.github/workflows/init_fork.yml:15`

### script-injection (severity: high)

Rule (a): ${{ ... }} expressions are directly interpolated inside run: shell command strings: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"`, `curl ... --header "Authorization: Bearer ${{ github.token }}"`.

Locations:

- `.github/workflows/test_package_repositories.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 8 files:

1. actions/instrument/deploy/action.yml: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions from run: shell blocks into env: blocks. Fixed: Find self step (with github-env-injection sanitization using tr -d '\n\r'), Determine repository, Determine version, Canonicalize, Find workflow/check-suite/repository-level observability steps, Deploy workflow/check-suite/repository-level instrumentation and startup optimization steps, Update workflow-level triggers, Deploy Copilot Setup, Deploy job-level instrumentation, Configure secret redaction, Modify permissions, Restore blank lines, Push (git commit -m), Enable auto-merge (gh pr merge).

2. .github/workflows/autobackport.yml: Fixed github.sha and github.event_path in dynamic step; matrix.ref in prepare_branch, format-patch/cherry-pick, and prepare_pr steps; steps.open-pr.outputs.pull-request-number in gh pr merge.

3. .github/workflows/build.yml: Fixed matrix.architecture in build-http step (all docker pull/run references); steps.determine-minimum-version.outputs.version in echo to file; needs.list-python-versions.outputs.versions in printf.

4. .github/workflows/publish.yml: Fixed github.event_path and github.token in dynamic step; steps.version.outputs.version, github.token, github.actor, and matrix.ref in docker login/push step.

5. .github/workflows/test_shell.yml: Fixed github.token and matrix.version in upgrade step; matrix.image in system-upgrade and install-manual-os steps; matrix.image in dependency step; matrix.image/update/shell in linux-shell step; matrix.shell in windows-shell step; github.token and matrix.version in performance steps.

6. .github/workflows/renovate.yml: Fixed all three gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }} occurrences.

7. .github/workflows/recompile_agentic_workflows.yml: Fixed gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}.

8. .github/workflows/refresh_demos.yml: Fixed github.token in curl/wget, matrix.demo_directory in cd commands, and github.token in sed command.

9. .github/workflows/init_fork.yml: Fixed secrets.ACTIONS_GITHUB_TOKEN in both run: blocks.

10. .github/workflows/test_package_repositories.yml: Fixed github.repository_owner and github.event.repository.name in echo/tee command; github.token in curl command.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings:
1. build.yml: Moved ${{ matrix.version }} to PYTHON_VERSION env var in the run step that checks python site-packages symlinks.
2. publish.yml: Moved ${{ steps.version.outputs.version }} to VERSION env var in the git tagging run step.
3. test_shell.yml: Moved ${{ secrets.DOCKERHUB_USERNAME }} and ${{ secrets.DOCKERHUB_TOKEN }} to DOCKERHUB_USERNAME and DOCKERHUB_TOKEN env vars in the docker login step.
4. test_github.yml (job-io-1): Moved ${{ steps.my-step.outputs.foo }} to MY_STEP_FOO env var.
5. test_github.yml (job-io-2): Moved ${{ needs.job-io-1.outputs.foo }} to JOB_IO_1_FOO env var.
6. test_github.yml (workflow-smoke): Moved ${{ secrets.GITHUB_TOKEN }} to GITHUB_TOKEN_VALUE env var in curl commands; moved workflow_run outputs to WORKFLOW_RUN_ID and WORKFLOW_RUN_ATTEMPT env vars in assertions.
7. test_github.yml (workflow job): Same fixes as workflow-smoke, plus moved gh_artifact_download call to use env vars.
8. test_github.yml (checksuite-smoke): Moved ${{ secrets.GITHUB_TOKEN }} to GITHUB_TOKEN_VALUE env var; moved ${{ steps.check_suite.outputs.id }} to CHECK_SUITE_ID env var.
9. test_github.yml (deploy job config step): Moved TEST_GITHUB_TOKEN, REPOSITORY_TEMPLATE, github.workflow, github.ref_name to env vars; also fixed github-env-injection by using printf '%s' ... | tr -d '\n\r' to sanitize the repository output value before writing to GITHUB_OUTPUT.
10. test_github.yml (deploy job): Fixed all remaining injections (DELETE curl, POST curl with matrix.private, git config, gh secret set + wget + yq commands, case statement with matrix.secret_redaction_strategy, final DELETE curl) by moving all ${{ }} expressions to env: blocks.

### Iteration 3

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/actions/instrument/deploy/action.yml:
1. 'Determine repository' step (line ~155): Replaced `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` with a while-read loop that sanitizes each line using `printf '%s' "$line" | tr -d '\n\r'` before writing `repository=$safe` to GITHUB_OUTPUT.
2. 'Determine version' step (line ~170): Same fix applied - replaced `xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"` with a while-read loop that sanitizes each line before writing `version=$safe` to GITHUB_OUTPUT.
Both fixes prevent caller-controlled newlines in ACTION_REPOSITORY and ACTION_VERSION inputs from injecting unexpected key=value pairs into GITHUB_OUTPUT.

