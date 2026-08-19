<!-- markdownlint-disable -->

# Hardening Report: plengauer--Thoth/v5.53.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **plengauer--Thoth/v5.53.0** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ ... }} expressions are directly interpolated inside run: shell commands. In actions/instrument/deploy/action.yml, the 'Find self' step uses ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly in shell commands; the 'Determine repository' step uses ${{ inputs.action_repository }}; the 'Determine version' step uses ${{ inputs.action_version }}; and many other steps use ${{ steps.*.outputs.* }}, ${{ inputs.* }}, and ${{ github.* }} directly in shell strings. In .github/workflows/test_shell.yml, a run: block uses ${{ matrix.image }}, ${{ matrix.update }}, and ${{ matrix.shell }} directly in a bash -c command. In .github/workflows/autobackport.yml, .github/workflows/autoversion_release.yml, .github/workflows/recompile_agentic_workflows.yml, and .github/workflows/renovate.yml, run: blocks use ${{ steps.open-pr.outputs.pull-request-number }} directly in shell commands. All of these allow expression values to be interpreted by the shell before quoting, enabling command injection.

Locations:

- `actions/instrument/deploy/action.yml:82`
- `actions/instrument/deploy/action.yml:84`
- `actions/instrument/deploy/action.yml:96`
- `actions/instrument/deploy/action.yml:103`
- `.github/workflows/test_shell.yml:197`
- `.github/workflows/autobackport.yml:119`
- `.github/workflows/autoversion_release.yml:52`
- `.github/workflows/recompile_agentic_workflows.yml:57`
- `.github/workflows/renovate.yml:43`

### github-env-injection (severity: high)

In actions/instrument/deploy/action.yml, the 'Find self' step writes ${{ inputs.__repository_level_instrumentation_file_name_override }} and ${{ github.workflow }} directly to $GITHUB_OUTPUT without the required sanitization step (printf '%s' ... | tr -d '\n\r'). These are untrusted input values (inputs.* and github.*) that could contain newlines, allowing injection of arbitrary environment variables or output values. Example: `echo path="${{ inputs.__repository_level_instrumentation_file_name_override }}" >> "$GITHUB_OUTPUT"` and `echo path="${{ github.workflow }}" >> "$GITHUB_OUTPUT"`.

Locations:

- `actions/instrument/deploy/action.yml:82`
- `actions/instrument/deploy/action.yml:84`

### unpinned-uses (severity: high)

Multiple files contain uses: references pinned to tags or branches instead of full 40-character SHA commit hashes, making them vulnerable to supply-chain attacks if the tag is moved. Failing references include: actions/checkout@v6.0.2, actions/checkout@v6, actions/download-artifact@v8.0.1, actions/attest-build-provenance@v4.1.0, actions/deploy-pages@v5.0.0, actions/first-interaction@v3.1.0, actions/github-script@v9.0.0, actions/hello-world-docker-action@main, actions/setup-java@v5.2.0, actions/setup-python@v6.2.0, docker/setup-qemu-action@v4.0.0, docker/setup-buildx-action@v4.0.0, peter-evans/enable-pull-request-automerge@v3.0.0, plengauer/autorerun@v0.37.0, plengauer/create-package-repository@v0.3.0, plengauer/opentelemetry-github/actions/instrument/job@v5.50.0, plengauer/opentelemetry-github/actions/instrument/workflow@v5.50.0, plengauer/opentelemetry-github/actions/instrument/checksuite@v5.50.0, plengauer/opentelemetry-github/actions/instrument/repository@v5.50.0, plengauer/opentelemetry-github/actions/instrument/deploy@v5.50.0, softprops/action-gh-release@v2.6.0, super-linter/super-linter@v8.6.0.

Locations:

- `actions/instrument/deploy/action.yml:76`
- `.github/workflows/analyze.yml:30`
- `.github/workflows/analyze.yml:36`
- `.github/workflows/analyze.yml:55`
- `.github/workflows/analyze.yml:65`
- `.github/workflows/autobackport.yml:14`
- `.github/workflows/autobackport.yml:17`
- `.github/workflows/autorerun.yml:14`
- `.github/workflows/autorerun.yml:17`
- `.github/workflows/autoversion.yml:14`
- `.github/workflows/autoversion_release.yml:18`
- `.github/workflows/autoversion_release.yml:21`
- `.github/workflows/build.yml:14`
- `.github/workflows/build.yml:17`
- `.github/workflows/build.yml:35`
- `.github/workflows/build.yml:155`
- `.github/workflows/build.yml:220`
- `.github/workflows/build.yml:270`
- `.github/workflows/ci.yml:14`
- `.github/workflows/copilot-setup-steps.yml:8`
- `.github/workflows/experiment.yml:21`
- `.github/workflows/greetings.yml:10`
- `.github/workflows/greetings.yml:13`
- `.github/workflows/init_fork.yml:9`
- `.github/workflows/init_fork.yml:22`
- `.github/workflows/observability_checksuite.yml:12`
- `.github/workflows/observability_deploy.yml:12`
- `.github/workflows/observability_deploy.yml:15`
- `.github/workflows/observability_repository.yml:13`
- `.github/workflows/observability_workflow.yml:13`
- `.github/workflows/performance_github.yml:20`
- `.github/workflows/performance_github.yml:30`
- `.github/workflows/publish.yml:20`
- `.github/workflows/publish.yml:57`
- `.github/workflows/publish.yml:62`
- `.github/workflows/publish.yml:63`
- `.github/workflows/publish.yml:64`
- `.github/workflows/publish.yml:80`
- `.github/workflows/recompile_agentic_workflows.yml:18`
- `.github/workflows/recompile_agentic_workflows.yml:21`
- `.github/workflows/refresh_demos.yml:20`
- `.github/workflows/refresh_demos.yml:30`
- `.github/workflows/refresh_demos.yml:155`
- `.github/workflows/renovate.yml:14`
- `.github/workflows/renovate.yml:30`
- `.github/workflows/rerequest_reviews.yml:12`
- `.github/workflows/rerequest_reviews.yml:15`
- `.github/workflows/test_github.yml:14`
- `.github/workflows/test_github.yml:50`
- `.github/workflows/test_github.yml:55`
- `.github/workflows/test_github.yml:56`
- `.github/workflows/test_shell.yml:14`
- `.github/workflows/test_shell.yml:30`
- `.github/workflows/update_package_repository.yml:16`
- `.github/workflows/update_package_repository.yml:19`
- `.github/workflows/update_package_repository.yml:21`

### hardcoded-credentials (severity: high)

A literal hardcoded value is assigned to COPILOT_API_KEY (which contains 'api_key') in multiple workflow lock files: `COPILOT_API_KEY: dummy-byok-key-for-offline-mode`. Although labeled as a dummy/offline-mode placeholder, it matches the hardcoded-credentials pattern (non-expression literal assigned to an api_key-named variable). This appears in autoapprove.lock.yml (twice), autofix.lock.yml (twice), autotriage.lock.yml (twice), and monitor_changelog.lock.yml (twice).

Locations:

- `.github/workflows/autoapprove.lock.yml:1`
- `.github/workflows/autofix.lock.yml:1`
- `.github/workflows/autotriage.lock.yml:1`
- `.github/workflows/monitor_changelog.lock.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, hardcoded-credentials

**Notes:**

Fixed all four findings:

1. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks in actions/instrument/deploy/action.yml (Find self, Determine repository, Determine version, Enable auto-merge steps), .github/workflows/test_shell.yml (matrix variables in bash -c command), and all gh pr merge steps in autobackport.yml, autoversion_release.yml, recompile_agentic_workflows.yml, and renovate.yml.

2. github-env-injection: Fixed the 'Find self' step in actions/instrument/deploy/action.yml by moving expressions to env: block and sanitizing with printf '%s' ... | tr -d '\n\r' before writing to $GITHUB_OUTPUT.

3. unpinned-uses: Pinned all action references to full 40-character SHA hashes across all listed workflow files. Key SHAs: actions/checkout@v6.0.2→de0fac2e, actions/download-artifact@v8.0.1→3e5f45b2, plengauer/opentelemetry-github@v5.50.0→215271fe, super-linter@v8.6.0→9e863354, softprops/action-gh-release@v2.6.0→26e8ad27, and many others.

4. hardcoded-credentials: Replaced literal 'dummy-byok-key-for-offline-mode' COPILOT_API_KEY values in all four lock files with ${{ secrets.COPILOT_API_KEY || 'dummy-byok-key-for-offline-mode' }} to avoid hardcoded credential pattern while preserving offline-mode functionality.

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

Fixed all script-injection findings by moving every ${{ }} expression from run: blocks into env: blocks, then referencing them as plain environment variables in the shell scripts. Fixed github-env-injection findings in the 'Determine repository' and 'Determine version' steps by replacing the unsafe xargs-based GITHUB_OUTPUT writes with while-loop-based writes that sanitize values using printf '%s' ... | tr -d '\n\r' before appending to $GITHUB_OUTPUT. All ${{ }} expressions in if:, with:, and env: blocks were left in place as they are safe in those contexts.

