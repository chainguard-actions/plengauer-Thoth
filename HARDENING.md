<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.55.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **plengauer--Thoth/v5.55.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in `actions/instrument/deploy/action.yml` directly interpolate `${{ }}` expressions inside shell command strings (rule a), allowing script injection. Examples include:
- **Find self step**: `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]` and `elif [ -r "${{ github.workflow }}" ]` and `ls "${{ inputs.workflows_directory }}"/*.yaml` — untrusted inputs/github context injected directly into shell.
- **Determine repository step**: `if [ -n "${{ inputs.action_repository }}" ]` and `cat "${{ steps.find-self.outputs.path }}"` — step output and input directly in shell.
- **Determine version step**: `if [ "${{ inputs.action_version }}" = same ]` — input directly in shell comparison.
- **Canonicalize step**: `sed -i 's~'"$repository"'~${{ steps.determine-repository.outputs.repository }}~g'` — step output directly in sed command.
- **Push step**: `git commit -m "${{ inputs.commit_message }}"` — user-controlled commit message injected directly into git command.
- **Enable auto-merge step**: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` — step output directly in shell command (unquoted).
- **Configure job-level instrumentation secret redaction step**: `job_level_instrumentation_secret_redaction_strategy="${{ inputs.job_level_instrumentation_secret_redaction_strategy }}"` — input directly in shell.
- Many additional steps use `${{ inputs.workflows_directory }}`, `${{ steps.find-*.outputs.path }}`, `${{ steps.determine-*.outputs.* }}`, `${{ github.repository }}`, `${{ github.repository_owner }}` directly in shell commands.

Locations:

- `actions/instrument/deploy/action.yml:88`
- `actions/instrument/deploy/action.yml:99`
- `actions/instrument/deploy/action.yml:114`
- `actions/instrument/deploy/action.yml:122`
- `actions/instrument/deploy/action.yml:430`
- `actions/instrument/deploy/action.yml:455`

### github-env-injection (severity: high)

Multiple `run:` blocks in `actions/instrument/deploy/action.yml` write values derived from untrusted inputs and GitHub context directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):
- **Find self step**: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` — user-controlled input written directly to GITHUB_OUTPUT.
- **Find self step**: `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"` — github context written directly to GITHUB_OUTPUT.
- **Determine version step**: `echo '${{ inputs.action_version }}' ... >> "$GITHUB_OUTPUT"` — user-controlled input written directly to GITHUB_OUTPUT.
- **Determine repository step**: `echo repository='{}' >> "$GITHUB_OUTPUT"` where the value is derived from `${{ steps.find-self.outputs.path }}` (itself tainted by inputs) — tainted step output written to GITHUB_OUTPUT.
- **Push step**: `echo "pushed=true" >> "$GITHUB_OUTPUT"` — these are literal values and are safe, but the same step uses `${{ inputs.commit_message }}` in the git command.
An attacker controlling any of these inputs could inject newlines to poison subsequent steps that read from GITHUB_OUTPUT.

Locations:

- `actions/instrument/deploy/action.yml:90`
- `actions/instrument/deploy/action.yml:92`
- `actions/instrument/deploy/action.yml:119`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection issues in actions/instrument/deploy/action.yml by:

1. **Find self step**: Moved `${{ inputs.__repository_level_instrumentation_file_name_override }}`, `${{ github.workflow }}`, and `${{ inputs.workflows_directory }}` to env block as REPO_LEVEL_OVERRIDE, GITHUB_WORKFLOW_NAME, WORKFLOWS_DIRECTORY. All GITHUB_OUTPUT writes now use `safe=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing.

2. **Determine repository step**: Moved `${{ github.action_repository }}`, `${{ steps.find-self.outputs.path }}`, `${{ inputs.action_repository }}` to env block. GITHUB_OUTPUT write sanitized with printf/tr.

3. **Determine version step**: Moved `${{ inputs.action_version }}`, `${{ steps.find-self.outputs.path }}`, `${{ steps.determine-repository.outputs.repository }}` to env block. GITHUB_OUTPUT write sanitized with printf/tr.

4. **Canonicalize step**: Added env block with WORKFLOWS_DIRECTORY and DETERMINED_REPOSITORY; replaced all ${{ }} in run block.

5. **Find workflow/check-suite/repository-level observability steps**: Added env blocks with WORKFLOWS_DIRECTORY; sanitized GITHUB_OUTPUT writes.

6. **Deploy workflow/check-suite/repository-level instrumentation steps**: Added env blocks with all needed variables.

7. **Deploy workflow/check-suite/repository-level startup optimization steps**: Added env blocks with FIND_*_PATH, WORKFLOWS_DIRECTORY, DETERMINED_VERSION, GITHUB_REPOSITORY_OWNER_VAR, GITHUB_TOKEN_INPUT.

8. **Update workflow-level observability triggers step**: Added comprehensive env block.

9. **Deploy Copilot Setup step**: Added WORKFLOWS_DIRECTORY env var.

10. **Deploy job-level instrumentation step**: Added comprehensive env block.

11. **Configure job-level instrumentation secret redaction step**: Added env block with WORKFLOWS_DIRECTORY, DETERMINED_REPOSITORY, JOB_LEVEL_SECRET_REDACTION_STRATEGY; replaced all ${{ }} in run block including the error message.

12. **Modify permissions step**: Added WORKFLOWS_DIRECTORY env var.

13. **Restore blank lines step**: Added WORKFLOWS_DIRECTORY env var.

14. **Push step**: Moved `${{ inputs.commit_message }}` to env block as COMMIT_MESSAGE.

15. **Enable auto-merge step**: Moved `${{ steps.open-pr.outputs.pull-request-number }}` to env block as PR_NUMBER; added quotes around the variable.

