<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Download go-coverage-report' step directly interpolates ${{ inputs.version }} and ${{ inputs.sha256sum }} into the run: shell command string: `run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"`.

These input values are controlled by whoever calls this composite action. They flow through YAML template substitution before the shell ever sees them, so shell metacharacters in the values (e.g. `; rm -rf /`, backticks, `$(...)`) would be executed by the shell. The values must be passed via env: variables and those variables must be double-quoted in the script.

Locations:

- `action.yml:107`

### script-injection (severity: high)

Rule (a) violation: The 'Code coverage report' step directly interpolates ${{ github.repository }}, ${{ github.event.pull_request.number }}, and ${{ github.run_id }} into the run: shell command string: `run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}"`.

All three values flow through YAML template substitution before the shell processes them. github.repository and github.event.pull_request.number in particular can contain attacker-controlled content (e.g. a repository name with shell metacharacters, or a crafted PR number). These values must be passed via env: variables and those variables must be double-quoted in the script.

Locations:

- `action.yml:127`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:109`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha256sum }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:109`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all four script-injection findings in action.yml:
1. 'Download go-coverage-report' step: Moved ${{ inputs.version }} and ${{ inputs.sha256sum }} from the run: command string to env: variables (INPUT_VERSION, INPUT_SHA256SUM). Updated scripts/download-cli-tool.sh to read VERSION and SHA256SUM from these env vars instead of positional arguments ($1, $2).
2. 'Code coverage report' step: Moved ${{ github.repository }}, ${{ github.event.pull_request.number }}, and ${{ github.run_id }} from the run: command string to env: variables (INPUT_GITHUB_REPOSITORY, INPUT_GITHUB_PULL_REQUEST_NUMBER, INPUT_GITHUB_RUN_ID). Updated scripts/github-action.sh to read GITHUB_REPOSITORY, GITHUB_PULL_REQUEST_NUMBER, and GITHUB_RUN_ID from these env vars instead of positional arguments ($1, $2, $3).

