<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.56.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.56.0** was hardened automatically. 9 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in actions/instrument/deploy/action.yml directly interpolate ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions into shell commands without routing through env: variables. Examples include: (1) 'Find self' step: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `ls "${{ inputs.workflows_directory }}"/*.yaml`; (2) 'Determine repository' step: `cat "${{ steps.find-self.outputs.path }}"` and `if [ -n "${{ inputs.action_repository }}" ]`; (3) 'Determine version' step: `if [ "${{ inputs.action_version }}" = same ]`; (4) 'Push' step: `git commit -m "${{ inputs.commit_message }}"`; (5) 'Enable auto-merge' step: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. These allow an attacker who controls input values to inject arbitrary shell commands.

Locations:

- `actions/instrument/deploy/action.yml:85`
- `actions/instrument/deploy/action.yml:87`
- `actions/instrument/deploy/action.yml:89`
- `actions/instrument/deploy/action.yml:103`
- `actions/instrument/deploy/action.yml:105`
- `actions/instrument/deploy/action.yml:113`
- `actions/instrument/deploy/action.yml:116`

### github-env-injection (severity: high)

Multiple run: blocks in actions/instrument/deploy/action.yml write values derived from untrusted inputs directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically: (1) 'Find self' step writes `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`; (2) 'Determine repository' step pipes untrusted values to `echo repository='{}' >> "$GITHUB_OUTPUT"`; (3) 'Determine version' step pipes `${{ inputs.action_version }}` to `echo version='{}' >> "$GITHUB_OUTPUT"`. An attacker can inject newlines into these values to set arbitrary environment variables or outputs.

Locations:

- `actions/instrument/deploy/action.yml:85`
- `actions/instrument/deploy/action.yml:87`
- `actions/instrument/deploy/action.yml:91`
- `actions/instrument/deploy/action.yml:116`
- `actions/instrument/deploy/action.yml:122`

### script-injection (severity: high)

The build.yml workflow directly interpolates ${{ matrix.architecture }}, ${{ matrix.version }}, and ${{ steps.*.outputs.* }} expressions into run: shell commands. Example: `debian_architecture="$(echo ${{ matrix.architecture }} | cut -d / -f 1 ...)"` and `sudo docker pull --platform linux/${{ matrix.architecture }} docker.io/"$(echo ${{ matrix.architecture }} | tr -d /)"`. Also: `echo "${{ steps.determine-minimum-version.outputs.version }}" > version`. These expressions are substituted before the shell parses the command, allowing injection of shell metacharacters.

Locations:

- `.github/workflows/build.yml:54`
- `.github/workflows/build.yml:56`
- `.github/workflows/build.yml:248`

### github-env-injection (severity: high)

The build.yml workflow writes a value derived from ${{ matrix.architecture }} directly to $GITHUB_OUTPUT without sanitization: `... | xargs -0 -I {} echo 'architecture={}' >> "$GITHUB_OUTPUT"`. The matrix value flows through the shell command and into the output file without newline stripping.

Locations:

- `.github/workflows/build.yml:57`

### script-injection (severity: high)

The autobackport.yml workflow directly interpolates ${{ github.sha }}, ${{ github.event_path }}, and ${{ matrix.ref }} expressions into run: shell commands. Examples: `git describe --tags --abbrev=0 ${{ github.sha }}`, `cat '${{ github.event_path }}'`, and `git log -1 --pretty=%s "${{ matrix.ref }}"`. These allow injection of shell metacharacters via attacker-controlled or workflow-controlled values.

Locations:

- `.github/workflows/autobackport.yml:30`
- `.github/workflows/autobackport.yml:31`
- `.github/workflows/autobackport.yml:75`

### github-env-injection (severity: high)

The autobackport.yml workflow writes values derived from ${{ matrix.ref }} directly to $GITHUB_OUTPUT without sanitization: `echo "$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"`, `echo author_name=... >> "$GITHUB_OUTPUT"`, and `echo author_email=... >> "$GITHUB_OUTPUT"`. The matrix.ref value is workflow-controlled and could contain newlines to inject additional output variables.

Locations:

- `.github/workflows/autobackport.yml:104`
- `.github/workflows/autobackport.yml:105`
- `.github/workflows/autobackport.yml:106`

### script-injection (severity: high)

The refresh_demos.yml workflow directly interpolates ${{ github.token }} into run: shell commands used in curl invocations: `curl --header "Authorization: Bearer ${{ github.token }}" ...` and `wget --header "Authorization: Bearer ${{ github.token }}" ...`. Any expression interpolated directly in a run: block is a script-injection risk as the value is substituted before the shell parses the command.

Locations:

- `.github/workflows/refresh_demos.yml:42`
- `.github/workflows/refresh_demos.yml:43`

### script-injection (severity: high)

The test_package_repositories.yml workflow directly interpolates ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} into run: shell commands: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"` and `--header "Authorization: Bearer ${{ github.token }}"`. These expressions are substituted before the shell parses the command.

Locations:

- `.github/workflows/test_package_repositories.yml:19`
- `.github/workflows/test_package_repositories.yml:28`

### script-injection (severity: high)

The init_fork.yml workflow directly interpolates ${{ secrets.ACTIONS_GITHUB_TOKEN }} into run: shell commands: `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]` and `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...`. Any ${{ }} expression interpolated directly in a run: block is substituted before the shell parses the command, bypassing shell quoting.

Locations:

- `.github/workflows/init_fork.yml:16`
- `.github/workflows/init_fork.yml:18`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 5 files:

1. actions/instrument/deploy/action.yml: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions from run: shell commands into env: blocks. Sanitized all GITHUB_OUTPUT writes with `printf '%s' "$VAR" | tr -d '\n\r'`.

2. .github/workflows/build.yml: Moved ${{ matrix.architecture }} to env var MATRIX_ARCHITECTURE in the build-http step, replaced all inline uses. Fixed GITHUB_OUTPUT write to use a sanitizing while loop. Moved ${{ steps.determine-minimum-version.outputs.version }} to env var.

3. .github/workflows/autobackport.yml: Moved ${{ github.sha }} and ${{ github.event_path }} to env vars in the setup/dynamic step. Moved ${{ matrix.ref }} to env vars in prepare_branch, cherry-pick, and prepare_pr steps. Sanitized all GITHUB_OUTPUT writes. Fixed gh pr merge step to use env var for PR number.

4. .github/workflows/refresh_demos.yml: Moved ${{ github.token }} to env var GITHUB_TOKEN_VAL in the curl/wget step.

5. .github/workflows/test_package_repositories.yml: Moved ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} to env vars.

6. .github/workflows/init_fork.yml: Moved ${{ secrets.ACTIONS_GITHUB_TOKEN }} to env var ACTIONS_TOKEN in both run steps.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed script injection vulnerabilities across 8 files by moving all ${{ }} expressions from run: shell commands to env: blocks and referencing them as plain environment variables. Files fixed: (1) actions/instrument/deploy/action.yml - fixed ~15 steps including Canonicalize, Find/Deploy workflow/check-suite/repository-level instrumentation, startup optimization, Update triggers, Deploy Copilot Setup, Deploy job-level instrumentation, Configure secret redaction, Modify permissions, Restore blank lines. (2) .github/workflows/publish.yml - fixed cat event_path, curl/docker login with github.token, steps.version.outputs.version, matrix.ref as docker tag. (3) .github/workflows/recompile_agentic_workflows.yml - fixed gh pr merge with steps.open-pr.outputs.pull-request-number. (4) .github/workflows/renovate.yml - fixed all 3 occurrences of gh pr merge with steps.open-pr.outputs.pull-request-number. (5) .github/workflows/test_shell.yml - fixed matrix.image/update/shell in run_tests_containerized.sh, matrix.version in mv/apt-get/timing commands. (6) .github/workflows/build.yml - fixed matrix.version in find command and needs.list-python-versions.outputs.versions in printf. (7) .github/workflows/test_github.yml - fixed steps.my-step.outputs.foo, secrets.GITHUB_TOKEN in curl, steps.config.outputs.user in git config, secrets.TEST_GITHUB_TOKEN in curl/gh, github.token/repository/sha/inputs.ref/matrix.secret_redaction_strategy in various commands. (8) .github/workflows/refresh_demos.yml - fixed github.token export, matrix.demo_directory in cd commands, github.token in sed.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in .github/workflows/test_shell.yml:
1. upgrade job: Moved ${{ github.token }} and ${{ matrix.version }} to an env: block, referencing them as $GITHUB_TOKEN and $MATRIX_VERSION in the shell script.
2. list-images job: Moved ${{ secrets.DOCKERHUB_USERNAME }} and ${{ secrets.DOCKERHUB_TOKEN }} to an env: block, referencing them as $DOCKERHUB_USERNAME and $DOCKERHUB_TOKEN in the shell script (also corrected quoting from single to double quotes around the variables).

### Iteration 4

**Fixes applied:** script-injection

**Notes:**

Fixed all script injection vulnerabilities in test_shell.yml and test_github.yml:

1. test_shell.yml - system-upgrade job: Moved `${{ matrix.image }}` to env block as MATRIX_IMAGE.
2. test_shell.yml - dependency job: Added MATRIX_IMAGE to existing env block, replaced direct interpolation.
3. test_shell.yml - install-manual-os job: Moved `${{ matrix.image }}` to new env block as MATRIX_IMAGE.
4. test_shell.yml - windows-shell job: Moved `${{ matrix.shell }}` to env block as MATRIX_SHELL, replaced all 3 direct interpolations with quoted `"$MATRIX_SHELL"` references.
5. test_github.yml - checksuite-smoke check_suite step: Moved `${{ secrets.GITHUB_TOKEN }}` to env block as GITHUB_TOKEN_VAL.
6. test_github.yml - checksuite-smoke validation step: Moved `${{ steps.check_suite.outputs.id }}` to env block as CHECK_SUITE_ID.
7. test_github.yml - workflow artifact download step: Moved `${{ secrets.GITHUB_TOKEN }}` and workflow_run output references to env block, replaced inline env var assignment with proper env var references.

### Iteration 5

**Fixes applied:** github-env-injection, script-injection

**Notes:**

Fixed three github-env-injection issues in actions/instrument/deploy/action.yml: the 'Find workflow-level observability', 'Find check-suite-level observability', and 'Find repository-level observability' steps now sanitize $workflow_file with `safe=$(printf '%s' "$workflow_file" | tr -d '\n\r')` before writing `path=$safe` to GITHUB_OUTPUT. Fixed one script-injection issue in .github/workflows/autoapprove.lock.yml: the gh api command now uses $GITHUB_PULL_REQUEST_NUMBER inside a properly double-quoted string instead of the unquoted concatenation pattern.

