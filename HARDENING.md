<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.59.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.59.0** was hardened automatically. 10 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: steps in actions/instrument/deploy/action.yml directly interpolate ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions inside shell commands, violating rule (a). Examples include: 'Find self' step uses ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} unquoted in shell conditions and echo commands; 'Determine repository' step uses ${{ inputs.action_repository }} in an if-condition; 'Determine version' step uses ${{ inputs.action_version }} in a case comparison; 'Canonicalize' step uses ${{ steps.determine-repository.outputs.repository }} in a sed command; 'Deploy workflow-level observability' and many subsequent steps embed ${{ inputs.workflows_directory }}, ${{ inputs.workflow_level_instrumentation_file_name }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.commit_message }}, ${{ steps.open-pr.outputs.pull-request-number }}, and others directly in run: shell scripts. Any caller of this composite action can inject arbitrary shell commands through these inputs.

Locations:

- `actions/instrument/deploy/action.yml:87`
- `actions/instrument/deploy/action.yml:97`
- `actions/instrument/deploy/action.yml:107`
- `actions/instrument/deploy/action.yml:113`
- `actions/instrument/deploy/action.yml:120`
- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:145`
- `actions/instrument/deploy/action.yml:165`
- `actions/instrument/deploy/action.yml:195`
- `actions/instrument/deploy/action.yml:220`
- `actions/instrument/deploy/action.yml:250`
- `actions/instrument/deploy/action.yml:280`
- `actions/instrument/deploy/action.yml:310`
- `actions/instrument/deploy/action.yml:340`

### script-injection (severity: high)

The 'autobackport.yml' workflow directly interpolates ${{ github.sha }}, ${{ github.event_path }}, and ${{ matrix.ref }} inside run: shell commands, violating rule (a). Specifically: the setup job's dynamic step uses ${{ github.sha }} unquoted in a git command and ${{ github.event_path }} in a cat command; the backport job's prepare_branch step uses ${{ matrix.ref }} unquoted in git describe and git format-patch commands; the prepare_pr step uses ${{ matrix.ref }} in git log commands and writes the result to $GITHUB_OUTPUT. The matrix.ref value is derived from commit IDs in the push event payload and could be attacker-influenced.

Locations:

- `.github/workflows/autobackport.yml:29`
- `.github/workflows/autobackport.yml:30`
- `.github/workflows/autobackport.yml:62`
- `.github/workflows/autobackport.yml:73`
- `.github/workflows/autobackport.yml:74`
- `.github/workflows/autobackport.yml:80`
- `.github/workflows/autobackport.yml:86`
- `.github/workflows/autobackport.yml:87`

### script-injection (severity: high)

The 'build.yml' workflow directly interpolates ${{ matrix.architecture }} inside run: shell commands, violating rule (a). The build-http job's run step uses ${{ matrix.architecture }} unquoted multiple times in curl, docker pull, and docker run commands (e.g., 'echo ${{ matrix.architecture }} | cut -d / -f 1', 'docker pull --platform linux/${{ matrix.architecture }}', 'docker run ... --platform linux/${{ matrix.architecture }}'). While matrix values are defined in the workflow, they still flow through YAML template substitution before the shell parses them.

Locations:

- `.github/workflows/build.yml:47`
- `.github/workflows/build.yml:49`
- `.github/workflows/build.yml:51`
- `.github/workflows/build.yml:53`
- `.github/workflows/build.yml:55`
- `.github/workflows/build.yml:60`
- `.github/workflows/build.yml:62`

### script-injection (severity: high)

The 'publish.yml' workflow directly interpolates ${{ github.token }}, ${{ github.actor }}, ${{ matrix.ref }}, and ${{ steps.version.outputs.version }} inside run: shell commands, violating rule (a). The setup job's dynamic step uses ${{ github.token }} in a curl Authorization header and ${{ github.event_path }} in a cat command. The publish job's docker login step uses 'echo ${{ github.token }} | sudo docker login ghcr.io -u ${{ github.actor }} --password-stdin' and uses ${{ matrix.ref }} and ${{ steps.version.outputs.version }} in for-loops and git tag commands.

Locations:

- `.github/workflows/publish.yml:30`
- `.github/workflows/publish.yml:31`
- `.github/workflows/publish.yml:79`
- `.github/workflows/publish.yml:82`
- `.github/workflows/publish.yml:86`
- `.github/workflows/publish.yml:100`
- `.github/workflows/publish.yml:104`

### script-injection (severity: high)

The 'test_shell.yml' workflow directly interpolates ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }}, and ${{ matrix.version }} inside run: shell commands, violating rule (a). The install-manual-os job uses ${{ matrix.image }} unquoted in a docker run command. The linux-shell job uses all three matrix values unquoted in 'bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'. The upgrade job uses ${{ matrix.version }} in apt-get install and performance measurement commands.

Locations:

- `.github/workflows/test_shell.yml:246`
- `.github/workflows/test_shell.yml:270`
- `.github/workflows/test_shell.yml:340`
- `.github/workflows/test_shell.yml:341`

### script-injection (severity: high)

The 'init_fork.yml' workflow directly interpolates ${{ secrets.ACTIONS_GITHUB_TOKEN }} inside run: shell commands, violating rule (a). Two run: steps use this secret expression directly in curl Authorization headers: 'curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...' and a second xargs-piped curl command. While secrets are masked in logs, interpolating them directly into run: scripts is a script-injection risk as the value flows through YAML template substitution before shell parsing.

Locations:

- `.github/workflows/init_fork.yml:18`
- `.github/workflows/init_fork.yml:19`

### script-injection (severity: high)

The 'test_package_repositories.yml' workflow directly interpolates ${{ github.repository_owner }} and ${{ github.event.repository.name }} inside a run: shell command, violating rule (a). The test-apt job's first step uses these values unquoted in an echo command that writes to /etc/apt/sources.list.d/otel.list: 'echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"'. The github.event.repository.name value is attacker-controllable.

Locations:

- `.github/workflows/test_package_repositories.yml:21`

### script-injection (severity: high)

The 'refresh_demos.yml' workflow directly interpolates ${{ matrix.demo_directory }} inside run: shell commands, violating rule (a). The generate job uses ${{ matrix.demo_directory }} unquoted in 'cd demos/${{ matrix.demo_directory }}' and 'sed -i s/${{ github.token }}/***/g otlp.json' commands. The matrix.demo_directory value is derived from directory names in the demos/ folder, but still flows through YAML template substitution before shell parsing.

Locations:

- `.github/workflows/refresh_demos.yml:55`
- `.github/workflows/refresh_demos.yml:68`
- `.github/workflows/refresh_demos.yml:100`
- `.github/workflows/refresh_demos.yml:101`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy/action.yml writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Specifically: 'echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"' and 'echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"'. A caller can inject newlines into these values to poison subsequent steps that read from GITHUB_OUTPUT.

Locations:

- `actions/instrument/deploy/action.yml:91`
- `actions/instrument/deploy/action.yml:93`

### github-env-injection (severity: high)

The 'prepare_pr' step in .github/workflows/autobackport.yml writes values derived from ${{ matrix.ref }} (a git commit SHA from the push event) to $GITHUB_OUTPUT without sanitization. Specifically: 'echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"' and 'echo author_name="$(git log -1 --pretty=%an "${{ matrix.ref }}" 2>/dev/null || echo "")" >> "$GITHUB_OUTPUT"'. Git commit titles and author names can contain newlines, allowing injection of arbitrary key=value pairs into GITHUB_OUTPUT.

Locations:

- `.github/workflows/autobackport.yml:86`
- `.github/workflows/autobackport.yml:87`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 8 files. (1) actions/instrument/deploy/action.yml: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions from run: blocks to env: blocks; added tr -d newlines sanitization for GITHUB_OUTPUT writes in Find self step. (2) .github/workflows/autobackport.yml: Moved ${{ github.sha }}, ${{ github.event_path }}, ${{ matrix.ref }} to env blocks; added tr -d newlines sanitization for GITHUB_OUTPUT writes in prepare_pr step. (3) .github/workflows/build.yml: Moved ${{ matrix.architecture }} to env block in build step. (4) .github/workflows/publish.yml: Moved ${{ github.event_path }}, ${{ github.token }}, ${{ github.actor }}, ${{ matrix.ref }}, ${{ steps.version.outputs.version }} to env blocks. (5) .github/workflows/test_shell.yml: Moved ${{ matrix.image }}, ${{ matrix.update }}, ${{ matrix.shell }}, ${{ matrix.version }}, ${{ github.token }} to env blocks across multiple jobs. (6) .github/workflows/init_fork.yml: Moved ${{ secrets.ACTIONS_GITHUB_TOKEN }} to env blocks. (7) .github/workflows/test_package_repositories.yml: Moved ${{ github.repository_owner }}, ${{ github.event.repository.name }}, ${{ github.token }} to env blocks. (8) .github/workflows/refresh_demos.yml: Moved ${{ github.token }} and ${{ matrix.demo_directory }} to env blocks; fixed sed command to use env var.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed 4 script-injection occurrences where `${{ steps.open-pr.outputs.pull-request-number }}` was directly interpolated into `run:` shell commands. In each case, moved the expression to the step's `env:` block as `PR_NUMBER: ${{ steps.open-pr.outputs.pull-request-number }}` and updated the shell command to use the quoted variable `"$PR_NUMBER"`. Files modified: `.github/workflows/recompile_agentic_workflows.yml` (1 fix) and `.github/workflows/renovate.yml` (3 fixes across renovate-package-dependency-python, renovate-test-images, and renovate-license jobs).

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 6 findings across 3 files:
1. test_shell.yml (list-images job, line ~131): Moved DOCKERHUB_USERNAME and DOCKERHUB_TOKEN from direct ${{ }} interpolation in run command to env block, referenced as quoted env vars.
2. test_shell.yml (windows-shell job, line ~461): Quoted $MATRIX_SHELL inside the bash -c string to prevent word splitting.
3. test_shell.yml (dependency job, line ~320): Changed heredoc delimiter from <<EOF to <<'EOF' to prevent outer shell expansion of $DEPENDENCY (variable is already passed to container via -e DEPENDENCY).
4. autoapprove.lock.yml (approve_pr job, line ~1498): Quoted $GITHUB_PULL_REQUEST_NUMBER by embedding it inside a double-quoted URL string.
5. actions/instrument/deploy/action.yml (determine-repository step, line ~120): Replaced xargs-based GITHUB_OUTPUT write with a sanitizing while loop using printf | tr -d '\n\r'.
6. actions/instrument/deploy/action.yml (determine-version step, line ~140): Replaced xargs-based GITHUB_OUTPUT write with a sanitizing while loop using printf | tr -d '\n\r'.

### Iteration 4

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection issues in build.yml and test_github.yml by moving ${{ }} expressions from run: shell blocks into step env: blocks, then referencing them as plain environment variables. Also fixed the github-env-injection in the deploy job's config step by adding proper sanitization with printf '%s' ... | tr -d '\n\r' before writing github.workflow and github.ref_name to GITHUB_OUTPUT. All expressions that were directly interpolated in shell commands (secrets.GITHUB_TOKEN, secrets.TEST_GITHUB_TOKEN, github.token, github.repository, github.sha, github.workflow, github.ref_name, steps.*.outputs.*, needs.*.outputs.*, matrix.*, env.REPOSITORY_TEMPLATE) have been moved to env blocks.

