<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.55.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.55.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `run:` blocks in the composite action directly interpolate `${{ inputs.* }}`, `${{ github.* }}`, and `${{ steps.*.outputs.* }}` expressions inside shell commands, enabling script injection. An attacker who controls input values (e.g., `inputs.commit_message`, `inputs.workflows_directory`, `inputs.action_version`, `inputs.job_level_instrumentation_exclude`, `github.workflow`, `github.repository_owner`, `steps.open-pr.outputs.pull-request-number`, etc.) can inject arbitrary shell commands. Affected steps include: 'Find self' (e.g., `if [ -r '${{ inputs.__repository_level_instrumentation_file_name_override }}' ]`), 'Determine repository' (e.g., `if [ -n "${{ inputs.action_repository }}" ]`), 'Determine version' (e.g., `if [ "${{ inputs.action_version }}" = same ]`), 'Canonicalize' (e.g., `sed -i 's~...~${{ steps.determine-repository.outputs.repository }}~g'`), 'Find workflow-level/check-suite/repository observability' (e.g., `ls "${{ inputs.workflows_directory }}"/*.yaml`), 'Deploy workflow-level/check-suite/repository-level observability' (multiple `${{ inputs.* }}` and `${{ steps.*.outputs.* }}`), 'Deploy workflow-level/check-suite/repository-level startup optimization' (e.g., `image=ghcr.io/${{ github.repository_owner }}/...`), 'Update workflow-level observability triggers' (multiple `${{ inputs.* }}`), 'Deploy Copilot Setup' (`${{ inputs.workflows_directory }}`), 'Deploy job-level instrumentation' (multiple `${{ inputs.* }}` and `${{ steps.*.outputs.* }}`), 'Configure job-level instrumentation secret redaction' (multiple `${{ inputs.* }}` and `${{ steps.*.outputs.* }}`), 'Modify permissions for job-level instrumentations' (`${{ inputs.workflows_directory }}`), 'Restore blank lines' (`${{ inputs.workflows_directory }}`), 'Push' (`git commit -m "${{ inputs.commit_message }}"`), and 'Enable auto-merge' (`gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`). All these violate sub-rule (a): `${{ ... }}` expressions must not appear directly inside `run:` shell command strings.

Locations:

- `actions/instrument/deploy/action.yml:93`
- `actions/instrument/deploy/action.yml:107`
- `actions/instrument/deploy/action.yml:118`
- `actions/instrument/deploy/action.yml:128`
- `actions/instrument/deploy/action.yml:135`
- `actions/instrument/deploy/action.yml:142`
- `actions/instrument/deploy/action.yml:149`
- `actions/instrument/deploy/action.yml:156`
- `actions/instrument/deploy/action.yml:165`
- `actions/instrument/deploy/action.yml:195`
- `actions/instrument/deploy/action.yml:230`
- `actions/instrument/deploy/action.yml:280`
- `actions/instrument/deploy/action.yml:330`
- `actions/instrument/deploy/action.yml:380`
- `actions/instrument/deploy/action.yml:430`
- `actions/instrument/deploy/action.yml:480`
- `actions/instrument/deploy/action.yml:530`
- `actions/instrument/deploy/action.yml:580`
- `actions/instrument/deploy/action.yml:630`
- `actions/instrument/deploy/action.yml:680`
- `actions/instrument/deploy/action.yml:730`

### github-env-injection (severity: high)

Several `run:` blocks in the composite action write values derived from untrusted inputs directly to `$GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). (1) The 'Find self' step writes `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"` — both are direct writes of unsanitized untrusted input. (2) The 'Determine version' step writes `echo '${{ inputs.action_version }}'` (via xargs) to `$GITHUB_OUTPUT` without sanitization. A newline in any of these values would allow injection of additional key=value pairs into the output file, potentially overwriting subsequent step outputs.

Locations:

- `actions/instrument/deploy/action.yml:95`
- `actions/instrument/deploy/action.yml:97`
- `actions/instrument/deploy/action.yml:120`

### unsafe-shell (severity: high)

The install script pipes remote content directly to a shell interpreter: `wget -O - https://raw.githubusercontent.com/plengauer/Thoth/main/INSTALL.sh | sh -e`. This pattern downloads and executes arbitrary code from a remote URL in a single pipeline without any integrity verification (no checksum, no signature check). If the remote URL is compromised or the content is tampered with in transit, malicious code will be executed on the runner.

Locations:

- `actions/instrument/shared/install.sh:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unsafe-shell

**Notes:**

1. script-injection: All ${{ }} expressions in run: blocks across all affected steps (Find self, Determine repository, Determine version, Canonicalize, Find workflow-level/check-suite/repository observability, Deploy workflow-level/check-suite/repository-level observability, Deploy workflow-level/check-suite/repository-level startup optimization, Update workflow-level observability triggers, Deploy Copilot Setup, Deploy job-level instrumentation, Configure job-level instrumentation secret redaction, Modify permissions for job-level instrumentations, Restore blank lines, Push, Enable auto-merge) have been moved to env: blocks. Shell scripts now reference $VAR_NAME environment variables instead. 2. github-env-injection: The Find self step now sanitizes INPUT_FILE_NAME_OVERRIDE and GITHUB_WORKFLOW_NAME with 'printf | tr -d \n\r' before writing to $GITHUB_OUTPUT. The Determine version step sanitizes INPUT_ACTION_VERSION similarly. 3. unsafe-shell: Replaced 'wget -O - URL | sh -e' with download-to-tempfile then execute pattern in install.sh.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Determine repository' step in actions/instrument/deploy/action.yml. Replaced `fi | xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` with `fi | { read -r _repo; safe=$(printf '%s' "$_repo" | tr -d '\n\r'); echo "repository=$safe" >> "$GITHUB_OUTPUT"; }`. This sanitizes newlines from the attacker-controlled values ($INPUT_ACTION_REPOSITORY from inputs.action_repository and $GITHUB_ACTION_REPOSITORY from github.action_repository) before writing to $GITHUB_OUTPUT, preventing newline injection attacks. The fix follows the same pattern already used in other steps in the same file.

