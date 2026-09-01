<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.61.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.61.5** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are interpolated directly inside run: shell commands in actions/instrument/deploy-local/action.yml. This includes ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, ${{ inputs.workflows_directory }}, ${{ inputs.action_version }}, ${{ inputs.action_repository }}, ${{ steps.find-self.outputs.path }}, ${{ steps.determine-repository.outputs.repository }}, ${{ steps.determine-instrumentation-version.outputs.version }}, ${{ inputs.workflow_level_instrumentation_workflow_name }}, ${{ inputs.workflow_level_instrumentation_exclude }}, ${{ inputs.job_level_instrumentation_secret_redaction_strategy }}, and many others. These are interpolated before the shell parses the script, enabling command injection by a caller supplying malicious input values.

Locations:

- `actions/instrument/deploy-local/action.yml:74`
- `actions/instrument/deploy-local/action.yml:77`
- `actions/instrument/deploy-local/action.yml:79`

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.commit_message }} is interpolated directly inside a run: shell command: `git commit -m "${{ inputs.commit_message }}"`. An attacker-controlled commit_message value can inject arbitrary shell commands. Additionally, `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}` interpolates a step output directly into the shell command.

Locations:

- `actions/instrument/deploy/action.yml:147`
- `actions/instrument/deploy/action.yml:168`

### script-injection (severity: high)

Sub-rule (a): ${{ secrets.ACTIONS_GITHUB_TOKEN }} is interpolated directly inside run: shell commands in two steps: `[ -n "${{ secrets.ACTIONS_GITHUB_TOKEN }}" ]` and `curl --fail -H "Authorization: Bearer ${{ secrets.ACTIONS_GITHUB_TOKEN }}" ...`. While secrets are masked in logs, direct interpolation into shell scripts is still a script-injection risk.

Locations:

- `.github/workflows/init_fork.yml:14`
- `.github/workflows/init_fork.yml:17`

### script-injection (severity: high)

Sub-rule (a): GitHub context expressions are interpolated directly inside run: shell commands. Specifically: `echo "deb [arch=all] https://${{ github.repository_owner }}.github.io/${{ github.event.repository.name }} stable main"` and `--header "Authorization: Bearer ${{ github.token }}"`. These values are substituted before the shell parses the script.

Locations:

- `.github/workflows/test_package_repositories.yml:20`
- `.github/workflows/test_package_repositories.yml:27`

### script-injection (severity: high)

Sub-rule (a): Matrix context expressions are interpolated directly inside a run: shell command: `bash -c 'cd tests && bash run_tests_containerized.sh "${{ matrix.image }}" "${{ matrix.update }}" "${{ matrix.shell }}"'`. Additionally, `opentelemetry-shell_${{ matrix.version }}_all.deb` and `sudo apt-get install -y ./opentelemetry-shell_${{ matrix.version }}_all.deb` interpolate matrix values directly into shell commands.

Locations:

- `.github/workflows/test_shell.yml:272`
- `.github/workflows/test_shell.yml:339`
- `.github/workflows/test_shell.yml:340`

### script-injection (severity: high)

Sub-rule (a): ${{ steps.open-pr.outputs.pull-request-number }} is interpolated directly inside a run: shell command: `gh pr merge --squash --auto ${{ steps.open-pr.outputs.pull-request-number }}`. Step outputs can be attacker-influenced.

Locations:

- `.github/workflows/autobackport.yml:93`
- `.github/workflows/recompile_agentic_workflows.yml:52`
- `.github/workflows/renovate.yml:88`
- `.github/workflows/renovate.yml:196`
- `.github/workflows/renovate.yml:222`

### github-env-injection (severity: high)

Multiple run: steps in actions/instrument/deploy-local/action.yml write values derived from ${{ inputs.* }} and ${{ github.* }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). Examples include: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"`, `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`, `echo version='{}' >> "$GITHUB_OUTPUT"` (where the version comes from ${{ inputs.action_version }}), and `echo "comment=# $comment" >> "$GITHUB_OUTPUT"` (where comment derives from ${{ inputs.action_version }}). An attacker-controlled input containing newlines can inject arbitrary key=value pairs into the GitHub output context.

Locations:

- `actions/instrument/deploy-local/action.yml:77`
- `actions/instrument/deploy-local/action.yml:79`
- `actions/instrument/deploy-local/action.yml:97`
- `actions/instrument/deploy-local/action.yml:107`
- `actions/instrument/deploy-local/action.yml:116`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection and github-env-injection findings across 7 files:

1. actions/instrument/deploy-local/action.yml: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions from run: shell commands to env: blocks across all steps (Find self, Determine repository, Determine version, Determine version comment, Canonicalize, Find workflow/check-suite/repository-level observability, Deploy workflow/check-suite/repository-level instrumentation and startup optimization, Update workflow-level triggers, Deploy Copilot Setup, Deploy job-level instrumentation, Configure secret redaction, Modify permissions, Annotate versions, Restore blank lines). Sanitized GITHUB_OUTPUT writes with printf '%s' | tr -d '\n\r' for github-env-injection fixes.

2. actions/instrument/deploy/action.yml: Moved ${{ inputs.commit_message }} to COMMIT_MESSAGE env var and ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var.

3. .github/workflows/init_fork.yml: Moved ${{ secrets.ACTIONS_GITHUB_TOKEN }} to ACTIONS_GITHUB_TOKEN env var.

4. .github/workflows/test_package_repositories.yml: Moved ${{ github.repository_owner }}, ${{ github.event.repository.name }}, and ${{ github.token }} to env vars.

5. .github/workflows/test_shell.yml: Moved matrix.image/update/shell to env vars for linux-shell job; moved matrix.version and github.token to env vars for performance job.

6. .github/workflows/autobackport.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var.

7. .github/workflows/recompile_agentic_workflows.yml: Moved ${{ steps.open-pr.outputs.pull-request-number }} to PR_NUMBER env var.

8. .github/workflows/renovate.yml: Fixed all 3 occurrences of ${{ steps.open-pr.outputs.pull-request-number }} by moving to PR_NUMBER env var.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/actions/instrument/deploy-local/action.yml:
1. 'Determine repository' step (~line 117): Replaced `| xargs -I '{}' echo repository='{}' >> "$GITHUB_OUTPUT"` with a pipe to a subshell that reads the value, strips newlines with `tr -d '\n\r'`, and writes the sanitized value to GITHUB_OUTPUT.
2. 'Determine version' step (~line 130): Same fix applied — replaced `| xargs -I '{}' echo version='{}' >> "$GITHUB_OUTPUT"` with the same sanitization pattern using `read -r` and `tr -d '\n\r'`.

