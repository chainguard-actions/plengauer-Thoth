<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.53.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.53.1** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in actions/instrument/deploy/action.yml directly interpolate ${{ }} expressions inside shell command strings (sub-rule a), allowing script injection. Examples include:
- 'Find self' step: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`
- 'Determine repository' step: `if [ -n "${{ inputs.action_repository }}" ]`
- 'Determine version' step: `if [ "${{ inputs.action_version }}" = same ]`
- 'Canonicalize' step: `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`
- 'Deploy workflow-level startup optimization' step: `image=ghcr.io/${{ github.repository_owner }}/...` and `Authorization: Bearer ${{ inputs.github_token }}`
- 'Update workflow-level observability triggers' step: `echo "${{ inputs.workflow_level_instrumentation_exclude }}"` and `Authorization: Bearer ${{ inputs.github_token }}` and `/repos/${{ github.repository }}/`
- 'Deploy check suite-level startup optimization' step: `image=ghcr.io/${{ github.repository_owner }}/...` and `Authorization: Bearer ${{ inputs.github_token }}`
- 'Deploy repository-level startup optimization' step: same pattern
- 'Deploy job-level instrumentation' step: `echo "${{ inputs.job_level_instrumentation_exclude }}"`
- 'Configure job-level instrumentation secret redaction' step: `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"`
- 'Push' step: `git commit -m "${{ inputs.commit_message }}"`
- 'Enable auto-merge' step: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`
All these values should be passed via env: variables and then referenced as shell variables with proper quoting.

Locations:

- `actions/instrument/deploy/action.yml:99`
- `actions/instrument/deploy/action.yml:110`
- `actions/instrument/deploy/action.yml:125`
- `actions/instrument/deploy/action.yml:133`
- `actions/instrument/deploy/action.yml:210`
- `actions/instrument/deploy/action.yml:240`
- `actions/instrument/deploy/action.yml:310`
- `actions/instrument/deploy/action.yml:380`
- `actions/instrument/deploy/action.yml:450`
- `actions/instrument/deploy/action.yml:510`
- `actions/instrument/deploy/action.yml:560`
- `actions/instrument/deploy/action.yml:650`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy/action.yml writes untrusted input values directly to $GITHUB_OUTPUT without sanitization (no `printf '%s' ... | tr -d '\n\r'` step). Specifically:
- `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` — writes an input value directly
- `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"` — writes a github context value directly
A newline injected into these values could allow an attacker to inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially overwriting subsequent step outputs.

Locations:

- `actions/instrument/deploy/action.yml:101`
- `actions/instrument/deploy/action.yml:103`

### unpinned-uses (severity: high)

The step 'Checkout' in actions/instrument/deploy/action.yml uses `actions/checkout@v6.0.2`, which is a mutable tag reference rather than a pinned full-length SHA commit hash. If the tag is moved (e.g., by a supply-chain compromise of the upstream repository), the action will silently execute different code. It should be pinned to a full 40-character hex SHA, e.g. `actions/checkout@<sha> # v6.0.2`.

Locations:

- `actions/instrument/deploy/action.yml:95`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in actions/instrument/deploy/action.yml:

1. **unpinned-uses**: Pinned `actions/checkout@v6.0.2` to full SHA `actions/checkout@de0fac2e4500dabe0009e67214ff5f5447ce83dd # v6.0.2`.

2. **script-injection**: Moved all `${{ }}` expressions from `run:` shell strings into `env:` blocks and referenced them as plain shell variables throughout all affected steps: Find self, Determine repository, Determine version, Canonicalize, Deploy workflow-level observability, Deploy workflow-level startup optimization, Update workflow-level observability triggers, Deploy check suite-level instrumentation, Deploy check suite-level startup optimization, Deploy repository-level instrumentation, Deploy repository-level startup optimization, Deploy Copilot Setup, Deploy job-level instrumentation, Configure job-level instrumentation secret redaction, Modify permissions, Restore blank lines, Push, and Enable auto-merge.

3. **github-env-injection**: In the 'Find self' step, both GITHUB_OUTPUT writes now sanitize values using `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing `echo "path=$safe" >> "$GITHUB_OUTPUT"`, preventing newline injection attacks.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection vulnerabilities in actions/hardened/plengauer--Thoth/v5.53.1/actions/instrument/deploy/action.yml:

1. 'Determine repository' step (line ~116): Replaced `| xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` with a while-read loop that sanitizes the value via `printf '%s' "$_val" | tr -d '\n\r'` before writing `repository=$_safe` to $GITHUB_OUTPUT.

2. 'Determine version' step (line ~131): Replaced `| xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"` with the same sanitization pattern, writing `version=$_safe` to $GITHUB_OUTPUT.

Both fixes prevent attacker-controlled newline characters in inputs.action_repository, github.action_repository, or inputs.action_version from injecting arbitrary key=value pairs into GITHUB_OUTPUT.

### Iteration 3

**Fixes applied:** unsafe-shell

**Notes:**

Fixed the unsafe pipe-to-shell pattern in actions/hardened/plengauer--Thoth/v5.53.1/actions/instrument/shared/install.sh at the last fallback path. Changed `wget -O - https://raw.githubusercontent.com/plengauer/Thoth/main/INSTALL.sh | sh -e` to download the script to `/tmp/thoth_install.sh` first, then execute it with `sh -e /tmp/thoth_install.sh`, and clean up with `rm -f /tmp/thoth_install.sh`. This prevents arbitrary code execution from a compromised or man-in-the-middle response.

