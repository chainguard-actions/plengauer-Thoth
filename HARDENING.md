<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.53.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.53.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `actions/instrument/deploy/action.yml` directly interpolate `${{ inputs.* }}`, `${{ github.* }}`, and `${{ steps.*.outputs.* }}` expressions into shell commands (sub-rule a). This allows an attacker who controls those values to inject arbitrary shell commands. Affected steps include: 'Find self' (e.g. `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`), 'Determine repository' (e.g. `if [ -n "${{ inputs.action_repository }}" ]`), 'Determine version' (e.g. `if [ "${{ inputs.action_version }}" = same ]`), 'Canonicalize' (e.g. `ls "${{ inputs.workflows_directory }}"`), 'Find workflow/check-suite/repository-level observability' steps, 'Deploy workflow/check-suite/repository-level observability/startup optimization' steps, 'Update workflow-level observability triggers', 'Deploy Copilot Setup', 'Deploy job-level instrumentation', 'Configure job-level instrumentation secret redaction', 'Push' (e.g. `git commit -m "${{ inputs.commit_message }}"`), and 'Enable auto-merge' (e.g. `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`).

Locations:

- `actions/instrument/deploy/action.yml:104`
- `actions/instrument/deploy/action.yml:113`
- `actions/instrument/deploy/action.yml:120`
- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:137`
- `actions/instrument/deploy/action.yml:144`
- `actions/instrument/deploy/action.yml:150`
- `actions/instrument/deploy/action.yml:157`
- `actions/instrument/deploy/action.yml:164`
- `actions/instrument/deploy/action.yml:171`
- `actions/instrument/deploy/action.yml:178`
- `actions/instrument/deploy/action.yml:185`
- `actions/instrument/deploy/action.yml:192`
- `actions/instrument/deploy/action.yml:199`
- `actions/instrument/deploy/action.yml:206`
- `actions/instrument/deploy/action.yml:213`
- `actions/instrument/deploy/action.yml:220`
- `actions/instrument/deploy/action.yml:227`
- `actions/instrument/deploy/action.yml:234`

### github-env-injection (severity: high)

Multiple `run:` blocks in `actions/instrument/deploy/action.yml` write values derived from untrusted inputs directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). In the 'Find self' step, `${{ inputs.__repository_level_instrumentation_file_name_override }}` and `${{ github.workflow }}` are written directly: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`. In the 'Determine repository' step, `${{ inputs.action_repository }}` flows into `$GITHUB_OUTPUT` via `xargs`. In the 'Determine version' step, `${{ inputs.action_version }}` flows into `$GITHUB_OUTPUT` via `xargs`. A newline in any of these values can inject arbitrary environment variables or output entries.

Locations:

- `actions/instrument/deploy/action.yml:105`
- `actions/instrument/deploy/action.yml:107`
- `actions/instrument/deploy/action.yml:120`
- `actions/instrument/deploy/action.yml:130`

### unpinned-uses (severity: high)

The `uses:` reference `actions/checkout@v6.0.2` in `actions/instrument/deploy/action.yml` uses a version tag instead of a full 40-character commit SHA. This means the action could be changed by the upstream repository without notice, creating a supply-chain risk. It should be pinned to a specific commit SHA (e.g. `actions/checkout@<40-char-sha> # v6.0.2`).

Locations:

- `actions/instrument/deploy/action.yml:97`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection

**Notes:**

Fixed all three findings in actions/instrument/deploy/action.yml:
1. Pinned actions/checkout@v6.0.2 to SHA de0fac2e4500dabe0009e67214ff5f5447ce83dd.
2. Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions from run: blocks into env: blocks for all affected steps (Find self, Determine repository, Determine version, Canonicalize, Find workflow/check-suite/repository-level observability, Deploy workflow/check-suite/repository-level observability/startup optimization, Update workflow-level observability triggers, Deploy Copilot Setup, Deploy job-level instrumentation, Configure job-level instrumentation secret redaction, Push, Enable auto-merge). Shell scripts now reference plain environment variables.
3. Sanitized all values written to $GITHUB_OUTPUT using printf '%s' "$VAR" | tr -d '\n\r' in the Find self, Determine repository, and Determine version steps to prevent newline injection.

