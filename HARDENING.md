<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.55.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.55.0** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple run: blocks in actions/instrument/deploy/action.yml directly interpolate ${{ ... }} expressions inside shell commands. This allows an attacker who controls the calling workflow's inputs or GitHub context values to inject arbitrary shell commands.

Affected steps and representative offending lines:
- 'Find self': `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `elif [ -r "${{ github.workflow }}" ]`
- 'Determine repository': `uses="$(cat "${{ steps.find-self.outputs.path }}" | ...)"` and `if [ -n "${{ inputs.action_repository }}" ]`
- 'Determine version': `if [ "${{ inputs.action_version }}" = same ]`
- 'Canonicalize': `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`
- 'Deploy workflow-level startup optimization': `version=${{ steps.determine-instrumentation-version.outputs.version }}` and `image=ghcr.io/${{ github.repository_owner }}/...`
- 'Update workflow-level observability triggers': `grep -qvF "$(echo "${{ inputs.workflow_level_instrumentation_exclude }}" | ...)"` and API call with `${{ github.repository }}`
- 'Deploy check suite-level startup optimization': same version/image pattern
- 'Deploy repository-level startup optimization': same version/image pattern
- 'Deploy Copilot Setup': `if [ -r "${{ inputs.workflows_directory }}"/copilot-setup-steps.yml ]`
- 'Deploy job-level instrumentation': `${{ inputs.job_level_instrumentation_exclude }}` and `${{ steps.determine-repository.outputs.repository }}`
- 'Configure job-level instrumentation secret redaction': `${{ inputs.job_level_instrumentation_secret_redaction_strategy }}`
- 'Push': `git commit -m "${{ inputs.commit_message }}"`
- 'Enable auto-merge': `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`

Locations:

- `actions/instrument/deploy/action.yml:104`
- `actions/instrument/deploy/action.yml:106`
- `actions/instrument/deploy/action.yml:115`
- `actions/instrument/deploy/action.yml:117`
- `actions/instrument/deploy/action.yml:127`
- `actions/instrument/deploy/action.yml:143`
- `actions/instrument/deploy/action.yml:200`
- `actions/instrument/deploy/action.yml:225`
- `actions/instrument/deploy/action.yml:270`
- `actions/instrument/deploy/action.yml:295`
- `actions/instrument/deploy/action.yml:320`
- `actions/instrument/deploy/action.yml:345`
- `actions/instrument/deploy/action.yml:375`
- `actions/instrument/deploy/action.yml:395`
- `actions/instrument/deploy/action.yml:430`
- `actions/instrument/deploy/action.yml:530`
- `actions/instrument/deploy/action.yml:560`

### github-env-injection (severity: high)

Multiple run: blocks write values derived from untrusted inputs directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r').

1. 'Find self' step: writes `${{ inputs.__repository_level_instrumentation_file_name_override }}` and `${{ github.workflow }}` directly to $GITHUB_OUTPUT via `echo path="${{ ... }}" >> "$GITHUB_OUTPUT"`. A newline in these values would inject additional key=value pairs into the output.

2. 'Determine version' step: pipes `echo '${{ inputs.action_version }}'` through xargs and writes `version=<value>` to $GITHUB_OUTPUT. A newline in inputs.action_version would inject additional entries.

3. 'Determine repository' step: pipes shell output (which includes `${{ inputs.action_repository }}`) through xargs and writes `repository=<value>` to $GITHUB_OUTPUT without sanitization.

Locations:

- `actions/instrument/deploy/action.yml:105`
- `actions/instrument/deploy/action.yml:107`
- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:138`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings in actions/instrument/deploy/action.yml:

1. **script-injection**: Moved all ${{ ... }} expressions from run: shell blocks into step env: blocks for every affected step: 'Find self', 'Determine repository', 'Determine version', 'Canonicalize', 'Find workflow-level observability', 'Find check-suite-level observability', 'Find repository-level observability', 'Deploy workflow-level observability', 'Deploy workflow-level startup optimization', 'Update workflow-level observability triggers', 'Deploy check suite-level instrumentation', 'Deploy check suite-level startup optimization', 'Deploy repository-level instrumentation', 'Deploy repository-level startup optimization', 'Deploy Copilot Setup', 'Deploy job-level instrumentation', 'Configure job-level instrumentation secret redaction', 'Modify permissions for job-level instrumentations', 'Restore blank lines', 'Push', and 'Enable auto-merge'.

2. **github-env-injection**: Sanitized all GITHUB_OUTPUT writes that used untrusted values:
   - 'Find self' step: uses `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing path= to GITHUB_OUTPUT
   - 'Determine repository' step: uses `read -r val; safe=$(printf '%s' "$val" | tr -d '\n\r')` before writing repository= to GITHUB_OUTPUT
   - 'Determine version' step: uses `read -r val; safe=$(printf '%s' "$val" | tr -d '\n\r')` before writing version= to GITHUB_OUTPUT

All ${{ ... }} expressions in if:, with:, and env: blocks were left as-is since those are safe contexts.

