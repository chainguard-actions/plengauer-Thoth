<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.60.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.60.2** was hardened automatically. 9 finding(s) were identified and resolved across 4 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps in deploy-local/action.yml directly interpolate ${{ }} expressions inside shell commands. Examples include: 'if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]' (Find self step, line ~88), 'cat "${{ steps.find-self.outputs.path }}"' (Determine repository step), 'if [ "${{ inputs.action_version }}" = same ]' (Determine version step), 'ls "${{ inputs.workflows_directory }}"/*.yaml' (Canonicalize step), and many more throughout the file. These ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions are expanded by the template engine before the shell sees them, allowing injection of arbitrary shell metacharacters.

Locations:

- `actions/instrument/deploy-local/action.yml:88`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:110`
- `actions/instrument/deploy-local/action.yml:120`

### script-injection (severity: high)

Sub-rule (a): The 'Push' step in actions/instrument/deploy/action.yml directly interpolates ${{ inputs.commit_message }} inside a run: shell command: 'git commit -m "${{ inputs.commit_message }}"'. The 'Enable auto-merge' step also interpolates ${{ steps.open-pr.outputs.pull-request-number }} directly: 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}'. Both allow shell metacharacter injection.

Locations:

- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:155`

### script-injection (severity: high)

Sub-rule (a): Multiple run: steps in .github/workflows/autobackport.yml directly interpolate ${{ }} expressions. The 'dynamic' step uses: '[ -z "$(git describe --tags --abbrev=0 ${{ github.sha }})" ]' and 'cat '${{ github.event_path }}' | jq ...'. The 'prepare_branch' step uses: 'current_tag=$(git describe --tags --abbrev=0 ${{ matrix.ref }})'. An anonymous step uses: 'git format-patch -1 "${{ matrix.ref }}" --stdout' and 'git cherry-pick -n "${{ matrix.ref }}"'. The 'prepare_pr' step uses: 'git log -1 --pretty=%s "${{ matrix.ref }}"'. A final step uses: 'gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}'.

Locations:

- `.github/workflows/autobackport.yml:31`
- `.github/workflows/autobackport.yml:32`
- `.github/workflows/autobackport.yml:60`
- `.github/workflows/autobackport.yml:75`
- `.github/workflows/autobackport.yml:83`
- `.github/workflows/autobackport.yml:108`

### script-injection (severity: high)

Sub-rule (a): .github/workflows/recompile_agentic_workflows.yml has a run: step that directly interpolates ${{ steps.open-pr.outputs.pull-request-number }}: 'run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}'. The steps.*.outputs.* context is workflow-controllable and allows shell metacharacter injection.

Locations:

- `.github/workflows/recompile_agentic_workflows.yml:50`

### script-injection (severity: high)

Sub-rule (a): .github/workflows/renovate.yml has three run: steps that directly interpolate ${{ steps.open-pr.outputs.pull-request-number }}: 'run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}' (appears in renovate-package-dependency-python, renovate-test-images, and renovate-license jobs). The steps.*.outputs.* context is workflow-controllable.

Locations:

- `.github/workflows/renovate.yml:120`
- `.github/workflows/renovate.yml:200`
- `.github/workflows/renovate.yml:280`

### script-injection (severity: high)

Sub-rule (a): .github/workflows/test_shell.yml has run: steps that directly interpolate matrix context values. One step uses: 'bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"''. Other steps use: 'opentelemetry-shell_${{ matrix.version }}_all.deb' in package install commands. The matrix.* context is workflow-controllable.

Locations:

- `.github/workflows/test_shell.yml:310`
- `.github/workflows/test_shell.yml:380`
- `.github/workflows/test_shell.yml:382`

### script-injection (severity: high)

Sub-rule (a): .github/workflows/test_package_repositories.yml has run: steps that directly interpolate github context values. One step uses: 'echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"'. Another step uses: '--header "Authorization: Bearer ${{ github.token }}"'. The github.* context values are interpolated before the shell sees them.

Locations:

- `.github/workflows/test_package_repositories.yml:18`
- `.github/workflows/test_package_repositories.yml:25`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy-local/action.yml writes untrusted input values directly to $GITHUB_OUTPUT without sanitization: 'echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"' and 'echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"'. The 'Determine version' step writes '${{ inputs.action_version }}'-derived values to $GITHUB_OUTPUT. The 'Determine version comment' step writes '${{ inputs.action_version }}'-derived values to $GITHUB_OUTPUT. None of these use the required 'printf "%s" ... | tr -d "\n\r"' sanitization before writing.

Locations:

- `actions/instrument/deploy-local/action.yml:89`
- `actions/instrument/deploy-local/action.yml:91`
- `actions/instrument/deploy-local/action.yml:112`
- `actions/instrument/deploy-local/action.yml:122`

### github-env-injection (severity: high)

The 'prepare_pr' step in .github/workflows/autobackport.yml writes values derived from ${{ matrix.ref }} (via git log) to $GITHUB_OUTPUT without sanitization: 'echo commit_title="$(git log -1 --pretty=%s "${{ matrix.ref }}")" >> "$GITHUB_OUTPUT"', 'echo author_name="$(git log -1 --pretty=%an "${{ matrix.ref }}" ...)" >> "$GITHUB_OUTPUT"', and 'echo author_email="$(git log -1 --pretty=%ae "${{ matrix.ref }}" ...)" >> "$GITHUB_OUTPUT"'. Commit messages can contain newlines, enabling GITHUB_OUTPUT injection. No 'printf "%s" ... | tr -d "\n\r"' sanitization is applied.

Locations:

- `.github/workflows/autobackport.yml:83`
- `.github/workflows/autobackport.yml:84`
- `.github/workflows/autobackport.yml:85`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script injection and github-env-injection findings across 7 files:

1. actions/instrument/deploy-local/action.yml: Moved all ${{ }} expressions from run: blocks to env: blocks across all 16+ steps. Added printf '%s' | tr -d '\n\r' sanitization for values written to GITHUB_OUTPUT in Find self and Determine version comment steps.

2. actions/instrument/deploy/action.yml: Moved ${{ inputs.commit_message }} to COMMIT_MESSAGE env var in Push step; moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var in Enable auto-merge step.

3. .github/workflows/autobackport.yml: Moved ${{ github.sha }} and ${{ github.event_path }} to env vars in dynamic step; moved ${{ matrix.ref }} to MATRIX_REF env var in prepare_branch, cherry-pick, and prepare_pr steps; added printf sanitization for GITHUB_OUTPUT writes in prepare_pr; moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var.

4. .github/workflows/recompile_agentic_workflows.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var.

5. .github/workflows/renovate.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var in all three jobs (renovate-package-dependency-python, renovate-test-images, renovate-license).

6. .github/workflows/test_shell.yml: Moved matrix.image, matrix.update, matrix.shell, matrix.version, github.token, secrets.DOCKERHUB_USERNAME, secrets.DOCKERHUB_TOKEN to env vars across linux-shell, windows-shell, upgrade, list-images, install-manual-os, system-upgrade, dependency, and performance jobs.

7. .github/workflows/test_package_repositories.yml: Moved ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} to env vars.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection findings by moving ${{ ... }} expressions from run: shell command strings into step env: blocks and referencing them as plain environment variables. Fixed github-env-injection findings in deploy-local/action.yml by replacing xargs-based GITHUB_OUTPUT writes with while-loop-based writes that sanitize values using printf '%s' ... | tr -d '\n\r'. Files modified: .github/workflows/build.yml, .github/workflows/init_fork.yml, .github/workflows/publish.yml, .github/workflows/refresh_demos.yml, .github/workflows/test_github.yml, actions/instrument/deploy-local/action.yml.

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed 3 issues across 2 files: (1) build.yml line ~500: moved `${{ steps.determine-minimum-version.outputs.version }}` from inline `run:` shell string into an `env:` block as `MINIMUM_VERSION`, referenced as `$MINIMUM_VERSION` in the shell. (2) agentics-maintenance.yml ~line 200: sanitized `GH_AW_OPERATION` before writing to GITHUB_OUTPUT using `printf '%s' "$GH_AW_OPERATION" | tr -d '\n\r'`. (3) agentics-maintenance.yml ~line 300: sanitized `GH_AW_RUN_URL` before writing to GITHUB_OUTPUT using `printf '%s' "$GH_AW_RUN_URL" | tr -d '\n\r'`.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/action/.github/workflows/test_github.yml at the `config` step. Added newline sanitization for `$GITHUB_WORKFLOW_VAL` and `$GITHUB_REF_NAME_VAL` using `printf '%s' "$VAR" | tr -d '\n\r'` before writing the repository value to `$GITHUB_OUTPUT`. The sanitized values are stored in `safe_workflow` and `safe_ref_name` local variables, which are then used in the echo command instead of the raw environment variables.

