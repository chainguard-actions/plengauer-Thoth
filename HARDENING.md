<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.53.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.53.1** was hardened automatically. 3 finding(s) were identified and resolved across 5 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Find self' step in actions/instrument/deploy/action.yml directly interpolates ${{ inputs.__repository_level_instrumentation_file_name_override }}, ${{ github.workflow }}, and ${{ inputs.workflows_directory }} inside run: shell commands (rule a). Similarly, the 'Determine version' step interpolates ${{ inputs.action_version }}, ${{ steps.find-self.outputs.path }}, and ${{ steps.determine-repository.outputs.repository }}. Dozens of other steps throughout the file also interpolate ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions directly inside shell commands, enabling script injection via attacker-controlled input values.

Locations:

- `actions/instrument/deploy/action.yml:100`
- `actions/instrument/deploy/action.yml:115`
- `actions/instrument/deploy/action.yml:130`
- `actions/instrument/deploy/action.yml:145`
- `actions/instrument/deploy/action.yml:160`
- `actions/instrument/deploy/action.yml:175`

### github-env-injection (severity: high)

The 'Find self' step in actions/instrument/deploy/action.yml writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without sanitization (no printf '%s' ... | tr -d '\n\r' step). The 'Determine version' step writes ${{ inputs.action_version }} to $GITHUB_OUTPUT without sanitization. The 'Push' step writes ${{ inputs.commit_message }} to $GITHUB_OUTPUT without sanitization. These allow newline injection into the GitHub output environment file.

Locations:

- `actions/instrument/deploy/action.yml:103`
- `actions/instrument/deploy/action.yml:105`
- `actions/instrument/deploy/action.yml:120`

### unpinned-uses (severity: high)

Multiple workflow files and action files reference external actions using mutable version tags instead of immutable 40-character commit SHAs. Failing references include: plengauer/opentelemetry-github/actions/instrument/job@v5.50.0, actions/checkout@v6.0.2, actions/checkout@v6, actions/download-artifact@v8.0.1, super-linter/super-linter@v8.6.0, docker/setup-qemu-action@v4.0.0, docker/setup-buildx-action@v4.0.0, actions/attest-build-provenance@v4.1.0, softprops/action-gh-release@v2.6.0, plengauer/autorerun@v0.37.0, actions/github-script@v9.0.0, actions/first-interaction@v3.1.0, plengauer/create-package-repository@v0.3.0, actions/upload-pages-artifact (tag), actions/deploy-pages@v5.0.0, peter-evans/enable-pull-request-automerge@v3.0.0, actions/setup-dotnet@v5, Vampire/setup-wsl (tag), actions/upload-artifact (tag), and others across analyze.yml, autorerun.yml, autoversion.yml, autoversion_release.yml, autobackport.yml, build.yml, ci.yml, copilot-setup-steps.yml, experiment.yml, greetings.yml, init_fork.yml, observability_checksuite.yml, observability_deploy.yml, observability_repository.yml, observability_workflow.yml, performance_github.yml, publish.yml, recompile_agentic_workflows.yml, refresh_demos.yml, renovate.yml, rerequest_reviews.yml, test_github.yml, test_shell.yml, update_package_repository.yml, workflow_example.yml, autoapprove.lock.yml, autofix.lock.yml, autotriage.lock.yml, monitor_changelog.lock.yml, and actions/instrument/deploy/action.yml.

Locations:

- `.github/workflows/analyze.yml:36`
- `.github/workflows/autorerun.yml:14`
- `.github/workflows/autoversion.yml:14`
- `.github/workflows/autoversion_release.yml:14`
- `.github/workflows/autobackport.yml:14`
- `.github/workflows/build.yml:14`
- `.github/workflows/ci.yml:14`
- `.github/workflows/copilot-setup-steps.yml:7`
- `.github/workflows/experiment.yml:18`
- `.github/workflows/greetings.yml:12`
- `.github/workflows/init_fork.yml:12`
- `.github/workflows/observability_checksuite.yml:12`
- `.github/workflows/observability_deploy.yml:12`
- `.github/workflows/observability_repository.yml:14`
- `.github/workflows/observability_workflow.yml:12`
- `.github/workflows/performance_github.yml:20`
- `.github/workflows/publish.yml:18`
- `.github/workflows/recompile_agentic_workflows.yml:16`
- `.github/workflows/refresh_demos.yml:18`
- `.github/workflows/renovate.yml:12`
- `.github/workflows/rerequest_reviews.yml:12`
- `.github/workflows/test_github.yml:14`
- `.github/workflows/test_shell.yml:18`
- `.github/workflows/update_package_repository.yml:12`
- `.github/workflows/workflow_example.yml:12`
- `.github/workflows/autoapprove.lock.yml:1`
- `.github/workflows/autofix.lock.yml:1`
- `.github/workflows/autotriage.lock.yml:1`
- `.github/workflows/monitor_changelog.lock.yml:1`
- `actions/instrument/deploy/action.yml:96`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses

**Notes:**

Fixed all three findings:

1. script-injection: Moved all ${{ inputs.* }}, ${{ github.* }}, and ${{ steps.*.outputs.* }} expressions out of run: shell commands in actions/instrument/deploy/action.yml into env: blocks. All steps now reference plain environment variables in shell scripts.

2. github-env-injection: Fixed GITHUB_OUTPUT writes in the 'Find self' step to sanitize values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing. The 'Push' step's commit_message is now in an env var (INPUT_COMMIT_MESSAGE). The 'Enable auto-merge' step uses PR_NUMBER env var.

3. unpinned-uses: Pinned all mutable tag references to full 40-character commit SHAs across all workflow files (analyze.yml, autorerun.yml, autoversion.yml, autoversion_release.yml, autobackport.yml, build.yml, ci.yml, copilot-setup-steps.yml, experiment.yml, greetings.yml, init_fork.yml, observability_checksuite.yml, observability_deploy.yml, observability_repository.yml, observability_workflow.yml, performance_github.yml, publish.yml, recompile_agentic_workflows.yml, refresh_demos.yml, renovate.yml, rerequest_reviews.yml, test_github.yml, test_shell.yml, update_package_repository.yml, workflow_example.yml, autoapprove.lock.yml, autofix.lock.yml, autotriage.lock.yml) and actions/instrument/deploy/action.yml. All 22+ unique action references have been pinned.

### Iteration 2

**Fixes applied:** script-injection, hardcoded-credentials, unpinned-uses, github-env-injection

**Notes:**

Fixed all 16 findings across 12 files:

**script-injection fixes:**
- autobackport.yml: Moved github.event_path, matrix.ref (5 occurrences) to env: blocks
- publish.yml: Moved github.event_path, github.token to env: blocks; also fixed docker login and image tagging steps
- test_shell.yml: Moved github.token+matrix.version (upgrade step), matrix.image+update+shell (linux-shell step), matrix.shell (windows-shell step) to env: blocks; also fixed additional performance step injections
- build.yml: Moved matrix.architecture (build-http step), steps.determine-minimum-version.outputs.version, needs.list-python-versions.outputs.versions to env: blocks
- refresh_demos.yml: Moved github.token (curl/wget step), github.token+matrix.demo_directory (demo step), matrix.demo_directory (diff step) to env: blocks
- init_fork.yml: Moved secrets.ACTIONS_GITHUB_TOKEN to env: block
- autoversion_release.yml: Moved steps.open-pr.outputs.pull-request-number to env: block
- recompile_agentic_workflows.yml: Moved steps.open-pr.outputs.pull-request-number to env: block
- renovate.yml: Moved steps.open-pr.outputs.pull-request-number to env: blocks (3 occurrences)

**hardcoded-credentials fixes:**
- autoapprove.lock.yml: Replaced COPILOT_API_KEY literal with ${{ secrets.COPILOT_API_KEY || 'dummy-byok-key-for-offline-mode' }} (2 occurrences)
- autofix.lock.yml: Same fix (2 occurrences)
- autotriage.lock.yml: Same fix (2 occurrences)
- monitor_changelog.lock.yml: Same fix (2 occurrences)

**unpinned-uses fix:**
- test_github.yml: Pinned actions/hello-world-docker-action@main to @8bcd8e1af3c095561f1043123848fc8b2db0f189

**github-env-injection fixes:**
- actions/instrument/deploy/action.yml: Replaced xargs-based GITHUB_OUTPUT writes with sanitized writes using printf '%s' ... | tr -d '\n\r' for both 'Determine repository' and 'Determine version' steps

### Iteration 3

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all 13 script-injection findings and 1 github-env-injection finding across 3 workflow files:

1. init_fork.yml (line 18): Moved `${{ secrets.ACTIONS_GITHUB_TOKEN }}` to env block as ACTIONS_GITHUB_TOKEN_CHECK.

2. test_shell.yml (line 100): Moved `${{ secrets.DOCKERHUB_USERNAME }}` and `${{ secrets.DOCKERHUB_TOKEN }}` to env block for docker login step.

3. test_shell.yml (line 130): Moved `${{ matrix.image }}` to env block as MATRIX_IMAGE for system-upgrade job.

4. test_shell.yml (line 200): Added MATRIX_IMAGE env var for dependency job's docker run step.

5. test_shell.yml (line 230): Moved `${{ matrix.image }}` to env block as MATRIX_IMAGE for install-manual-os job.

6. test_shell.yml (line 350): Moved `${{ matrix.version }}` to env block as MATRIX_VERSION for performance job, using shell variable expansion in filenames.

7. test_github.yml (line 280): Moved `${{ steps.my-step.outputs.foo }}` to env block as MY_STEP_FOO for job-io-1.

8. test_github.yml (line 300): Moved `${{ needs.job-io-1.outputs.foo }}` to env block as JOB_IO_1_FOO for job-io-2.

9. test_github.yml (lines 350, 360, 365): Fixed workflow-smoke job - moved GITHUB_TOKEN to env block for curl step; moved workflow_run outputs to env block for validation step.

10. test_github.yml (lines 400, 415): Fixed checksuite-smoke job - moved GITHUB_TOKEN to env block for curl step; moved check_suite output to env block for validation step.

11. test_github.yml (lines 450, 465, 475): Fixed workflow job - same pattern as workflow-smoke; also moved INPUT_GITHUB_TOKEN and workflow_run outputs to env block for gh_artifact_download call.

12. test_github.yml (lines 520-580): Fixed deploy job - moved TEST_GITHUB_TOKEN, github context values, matrix values, and steps.config.outputs to env blocks for all affected steps. Also fixed github-env-injection by using printf '%s' | tr -d '\n\r' for the repository name construction.

### Iteration 4

**Fixes applied:** github-env-injection

**Notes:**

Fixed all four github-env-injection findings in `.github/workflows/autobackport.yml` in the `prepare_pr` step. Each value derived from `matrix.ref` that is written to `$GITHUB_OUTPUT` is now sanitized using `printf '%s' "$var" | tr -d '\n\r'` before being written:
1. `commit_title` → stored as `safe_commit_title` before writing
2. `reviewers` → stored as `safe_reviewers` before writing
3. `author_name` → stored as `safe_author_name` before writing
4. `author_email` → stored as `safe_author_email` before writing
This prevents newline injection attacks where attacker-controlled values (commit messages, author names/emails, reviewer logins) containing newline characters could inject additional key-value pairs into `$GITHUB_OUTPUT`.

### Iteration 5

**Fixes applied:** script-injection

**Notes:**

Fixed script injection in build-python-site-packages job at .github/workflows/build.yml. The `${{ matrix.version }}` expression was directly embedded in a `run:` shell command (`find -L src/.../venv/lib/python${{ matrix.version }}/site-packages ...`). Moved the expression into the step's `env:` block as `MATRIX_VERSION: ${{ matrix.version }}` and replaced the inline expression with `"$MATRIX_VERSION"` in the shell command. All other occurrences of `${{ matrix.version }}` in the file are in `with:` blocks and are not vulnerable to shell injection.

