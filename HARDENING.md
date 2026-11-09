<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.67.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.67.0** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

(a) Direct expression interpolation in the 'Push' step: `git commit -m "${{ inputs.commit_message }}"`. A commit message containing `"` or `$(...)` breaks out of the quotes and runs commands while GH_TOKEN (a write token) is in the environment. Pass it through env (e.g. COMMIT_MESSAGE) and use "$COMMIT_MESSAGE".

Locations:

- `actions/instrument/deploy/action.yml:196`

### script-injection (severity: high)

(a) Direct expression interpolation in the 'Enable auto-merge' step: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. The step output is put into the shell script unquoted. Move it to env and quote it.

Locations:

- `actions/instrument/deploy/action.yml:221`

### script-injection (severity: high)

(a) The 'Find self' step puts `${{ inputs.__repository_level_instrumentation_file_name_override }}`, `${{ github.workflow }}` and `${{ inputs.workflows_directory }}` directly into run: (e.g. `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`, `elif [ -r "${{ github.workflow }}" ]`). The workflow name and the inputs can contain quotes or command substitutions.

Locations:

- `actions/instrument/deploy-local/action.yml:96`

### script-injection (severity: high)

(a) The 'Determine repository' and 'Determine version' / 'Determine version comment' steps put `${{ steps.find-self.outputs.path }}`, `${{ inputs.action_repository }}`, `${{ inputs.action_version }}` and `${{ steps.determine-repository.outputs.repository }}` directly into run: (e.g. `if [ -n "${{ inputs.action_repository }}" ]; then echo '${{ inputs.action_repository }}'`, `if [ "${{ inputs.action_version }}" = same ]`, `comment='${{ inputs.action_version }}'`).

Locations:

- `actions/instrument/deploy-local/action.yml:111`
- `actions/instrument/deploy-local/action.yml:129`
- `actions/instrument/deploy-local/action.yml:139`

### script-injection (severity: high)

(a) The 'Canonicalize' and 'Find workflow/check-suite/repository-level observability' steps put `${{ inputs.workflows_directory }}` and `${{ steps.determine-repository.outputs.repository }}` directly into run: (e.g. `(ls "${{ inputs.workflows_directory }}"/*.yaml ...`, `sed -i 's~'"$repository"'~${{ steps.determine-repository.outputs.repository }}~g'`).

Locations:

- `actions/instrument/deploy-local/action.yml:146`

### script-injection (severity: high)

(a) The 'Deploy workflow-level observability', 'Deploy check suite-level instrumentation' and 'Deploy repository-level instrumentation' steps put `${{ inputs.workflow_level_instrumentation_workflow_name }}`, `${{ inputs.workflows_directory }}`, `${{ inputs.*_file_name }}`, `${{ steps.determine-repository.outputs.repository }}` and `${{ steps.determine-instrumentation-version.outputs.version }}` directly into run: (e.g. `echo 'name: ${{ inputs.workflow_level_instrumentation_workflow_name }}`).

Locations:

- `actions/instrument/deploy-local/action.yml:178`

### script-injection (severity: high)

(a) The 'Deploy workflow/check suite/repository-level startup optimization' steps put `${{ inputs.github_token }}`, `${{ github.repository_owner }}` and `${{ steps.determine-instrumentation-version.outputs.version }}` directly into run: (e.g. `version=${{ steps.determine-instrumentation-version.outputs.version }}` unquoted, `curl ... --header "Authorization: Bearer ${{ inputs.github_token }}"`).

Locations:

- `actions/instrument/deploy-local/action.yml:194`

### script-injection (severity: high)

(a) The 'Update workflow-level observability triggers' step puts `${{ inputs.workflow_level_instrumentation_exclude }}`, `${{ inputs.github_token }}`, `${{ github.repository }}` and several `${{ steps.*.outputs.path }}` values directly into run: (e.g. `[ -n "${{ inputs.workflow_level_instrumentation_exclude }}" ]`).

Locations:

- `actions/instrument/deploy-local/action.yml:212`

### script-injection (severity: high)

(a) The 'Deploy Copilot Setup', 'Deploy job-level instrumentation', 'Configure job-level instrumentation secret redaction', 'Modify permissions', 'Remove explicit trace context declarations', 'Annotate instrumentation action versions' and 'Restore blank lines' steps all put `${{ inputs.* }}` / `${{ steps.*.outputs.* }}` directly into run: (e.g. `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"`, `echo '::error ::Illegal secret redaction strategy "${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"!'`, `version='${{ steps.determine-instrumentation-version.outputs.version }}'`, `[ -n "${{ inputs.job_level_instrumentation_exclude }}" ]`).

Locations:

- `actions/instrument/deploy-local/action.yml:330`
- `actions/instrument/deploy-local/action.yml:360`
- `actions/instrument/deploy-local/action.yml:420`

### github-env-injection (severity: high)

The 'Find self' step writes `${{ inputs.__repository_level_instrumentation_file_name_override }}`, `${{ github.workflow }}` and file paths derived from `${{ inputs.workflows_directory }}` to $GITHUB_OUTPUT (`echo path="..." >> "$GITHUB_OUTPUT"`) without `printf '%s' ... | tr -d '\n\r'` sanitization first.

Locations:

- `actions/instrument/deploy-local/action.yml:97`

### github-env-injection (severity: high)

The 'Determine repository' step writes `${{ inputs.action_repository }}` / the inherited `$GITHUB_ACTION_REPOSITORY` / a value parsed from the workflow file to $GITHUB_OUTPUT via `xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` without newline sanitization.

Locations:

- `actions/instrument/deploy-local/action.yml:124`

### github-env-injection (severity: high)

The 'Determine version' step writes `${{ inputs.action_version }}` (or a value parsed from the workflow) to $GITHUB_OUTPUT via `xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"` without newline sanitization.

Locations:

- `actions/instrument/deploy-local/action.yml:133`

### github-env-injection (severity: high)

The 'Determine version comment' step writes `comment='${{ inputs.action_version }}'` (or a comment parsed from the workflow file) to $GITHUB_OUTPUT with `echo "comment=# $comment" >> "$GITHUB_OUTPUT"` without `printf '%s' | tr -d '\n\r'` sanitization.

Locations:

- `actions/instrument/deploy-local/action.yml:143`

### github-env-injection (severity: high)

The 'Find workflow-level / check-suite-level / repository-level observability' steps write file paths listed from the input-controlled `${{ inputs.workflows_directory }}` to $GITHUB_OUTPUT (`echo path="$workflow_file" >> "$GITHUB_OUTPUT"`) without newline sanitization.

Locations:

- `actions/instrument/deploy-local/action.yml:157`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Neither file was run after the edits. I only re-read deploy-local/action.yml to check that no ${{ }} expression is left inside a run: block, and checked the two deploy/action.yml edits by eye.

deploy/action.yml: the commit message now comes from the step's env as COMMIT_MESSAGE and is used as "$COMMIT_MESSAGE". The PR number in the auto-merge step comes from env as PR_NUMBER and is quoted.

deploy-local/action.yml: in every run: step, the expressions (inputs.*, github.workflow/repository/repository_owner, steps.*.outputs.*) now come from the step's env: block. In the shell they are quoted variables, and inside single-quoted yq or sed text they are spliced in with the '"$VAR"' pattern, so the yq expressions, sed patterns and generated workflow text come out the same as before. The token now reaches curl through INPUT_GITHUB_TOKEN instead of being pasted into the script.

Newlines are now stripped before writing to $GITHUB_OUTPUT. This covers the 'path=' outputs of Find self and the three Find steps, the repository and version outputs, and the version comment. The repository and version outputs used to be written through 'xargs -I {} echo key={}'; that is now a while-read loop that cleans each line and writes it. Two small differences: leading whitespace on a line is no longer trimmed the way xargs did, and a value containing a quote no longer makes xargs fail.

One quirk kept on purpose: upstream refers to a missing input, inputs.check_suite_instrumentation_file_name, which always evaluates to an empty string. I passed it through env unchanged so behaviour stays the same.

