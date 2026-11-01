<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.66.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.66.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): expressions are placed directly in run scripts in the deploy composite action. In the 'Push' step: `git commit -m "${{ inputs.commit_message }}"`. Anyone who controls commit_message can run commands by breaking out of the quotes. In the 'Enable auto-merge' step: `run: gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` (unquoted step output). Fix: pass each value through `env:` and use it as "$VAR".

Locations:

- `actions/instrument/deploy/action.yml:195`
- `actions/instrument/deploy/action.yml:220`

### script-injection (severity: high)

Rule (a): many run steps in the deploy-local composite action put `${{ }}` expressions straight into the shell script. Examples: 'Find self' has `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `elif [ -r "${{ github.workflow }}" ]`. 'Determine repository' has `uses="$(cat "${{ steps.find-self.outputs.path }}" ...` and `echo '${{ inputs.action_repository }}'`. 'Determine version' has `if [ "${{ inputs.action_version }}" = same ]` and `echo '${{ inputs.action_version }}'`. 'Determine version comment' has `comment='${{ inputs.action_version }}'`. 'Canonicalize' and the 'Find * observability' steps use `ls "${{ inputs.workflows_directory }}"/*.yaml` and `sed -i 's~'"$repository"'~${{ steps.determine-repository.outputs.repository }}~g'`. The later deploy steps also interpolate inputs.workflow_level_instrumentation_*, inputs.job_level_instrumentation_secret_redaction_strategy and steps.determine-repository.outputs.repository directly into echo/yq commands. Fix: move each value into `env:` and quote every expansion.

Locations:

- `actions/instrument/deploy-local/action.yml:97`
- `actions/instrument/deploy-local/action.yml:112`
- `actions/instrument/deploy-local/action.yml:130`
- `actions/instrument/deploy-local/action.yml:140`
- `actions/instrument/deploy-local/action.yml:149`

### github-env-injection (severity: high)

The deploy-local action writes values from untrusted inputs and github context to $GITHUB_OUTPUT without the `printf '%s' ... | tr -d '\n\r'` sanitization step. 'Find self': `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. 'Determine repository': `echo '${{ inputs.action_repository }}' ... | xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"`. 'Determine version': `echo '${{ inputs.action_version }}' | xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"`. 'Determine version comment': `echo "comment=# $comment" >> "$GITHUB_OUTPUT"`, where comment comes from inputs.action_version. A value containing a newline can inject extra step outputs.

Locations:

- `actions/instrument/deploy-local/action.yml:98`
- `actions/instrument/deploy-local/action.yml:100`
- `actions/instrument/deploy-local/action.yml:125`
- `actions/instrument/deploy-local/action.yml:134`
- `actions/instrument/deploy-local/action.yml:145`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

deploy/action.yml: commit_message and the PR number now go through env (COMMIT_MESSAGE, PR_NUMBER) and are quoted.

deploy-local/action.yml: every expression in every run step now goes through env, including inputs, github.workflow/repository/repository_owner, step outputs and github_token. In the yq/sed expressions, the values are joined into the single-quoted strings ('..."'"$ACTION_REPO"'/...'), so the shell never evaluates them. The files the steps generate are built the same way.

The GITHUB_OUTPUT writes in 'Find self', 'Determine repository', 'Determine version' and 'Determine version comment' now strip newlines with printf '%s' ... | tr -d '\n\r'. Two of these steps used to pipe into xargs -I. They now save the value, keep the last non-empty line, and write it only if it is not empty, which is what the xargs version did for normal values. One small difference: xargs used to trim leading spaces and treat quote characters specially, and the new version keeps the value exactly as given.

One reference to a non-existent input, inputs.check_suite_instrumentation_file_name, is in the original code. I moved it into env unchanged so the action still behaves the same way.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

deploy/action.yml: Push step now passes inputs.commit_message via env COMMIT_MESSAGE and uses git commit -m "$COMMIT_MESSAGE". deploy-local/action.yml: in Find self (else branch) and the three Find *-level observability steps, workflow_file is sanitized with printf '%s' | tr -d '\n\r' before being written to $GITHUB_OUTPUT, keeping the original && ... && break || true control flow.

