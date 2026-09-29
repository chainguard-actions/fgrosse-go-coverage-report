<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Direct expression interpolation in `run:` shell command. The 'Download go-coverage-report' step interpolates `${{ inputs.version }}` and `${{ inputs.sha256sum }}` directly into the shell command string: `run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"`.

An attacker-controlled value for `inputs.version` or `inputs.sha256sum` (e.g. containing shell metacharacters or newlines) is expanded by the YAML template engine before the shell ever sees it, enabling command injection. These values should be passed via `env:` variables and referenced as `"$VERSION"` / `"$SHA256SUM"` inside the script.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (a): Direct expression interpolation in `run:` shell command. The 'Code coverage report' step interpolates `${{ github.repository }}`, `${{ github.event.pull_request.number }}`, and `${{ github.run_id }}` directly into the shell command string: `run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}"`.

These GitHub context values are substituted by the YAML template engine before the shell parses the command. A malicious repository name or PR number containing shell metacharacters could enable command injection. These values should be passed via `env:` variables (e.g. `GITHUB_REPOSITORY`, `PR_NUMBER`, `RUN_ID`) and referenced as quoted shell variables inside the script.

Locations:

- `action.yml:124`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:106`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha256sum }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:106`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed two script injection vulnerabilities in hardened/action/action.yml:
1. 'Download go-coverage-report' step: Moved ${{ inputs.version }} and ${{ inputs.sha256sum }} from the run: command into env: variables (VERSION and SHA256SUM). The run: command now uses "$VERSION" and "$SHA256SUM" as shell variables.
2. 'Code coverage report' step: Moved ${{ github.repository }}, ${{ github.event.pull_request.number }}, and ${{ github.run_id }} from the run: command into env: variables (REPOSITORY, PR_NUMBER, RUN_ID). The run: command now uses "$REPOSITORY", "$PR_NUMBER", and "$RUN_ID" as shell variables.
All ${{ }} expressions now only appear in env: blocks and outputs: sections, preventing YAML template engine substitution from enabling command injection.

