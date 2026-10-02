<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.5.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download go-coverage-report' step directly interpolates ${{ inputs.version }} and ${{ inputs.sha256sum }} into the run: shell command string. These are caller-controlled inputs that flow through YAML template substitution before the shell sees them, enabling command injection. The offending line is: run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}". Fix: move the values into env: variables and reference them as quoted shell variables ("$VERSION" / "$SHA256SUM") in the script.

Locations:

- `action.yml:155`

### script-injection (severity: high)

Sub-rule (a): The 'Code coverage report' step directly interpolates ${{ github.repository }}, ${{ github.event.pull_request.number }}, and ${{ github.run_id }} into the run: shell command string. These values flow through YAML template substitution before the shell sees them, enabling command injection (e.g. a repository name or PR number containing shell metacharacters). The offending line is: run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}". Fix: move the values into env: variables (they are already partially present in the env: block) and pass them to the script via environment rather than positional arguments interpolated in the run: string.

Locations:

- `action.yml:163`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:146`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha256sum }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:146`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Fixed all four script-injection findings in action.yml:

1. 'Download go-coverage-report' step: Moved ${{ inputs.version }} and ${{ inputs.sha256sum }} from the run: command line into the step's env: block as VERSION and SHA256SUM. Updated scripts/download-cli-tool.sh to read VERSION from the environment (with positional arg fallback for backward compatibility).

2. 'Code coverage report' step: Moved ${{ github.repository }}, ${{ github.event.pull_request.number }}, and ${{ github.run_id }} from the run: command line into the step's env: block as GITHUB_REPOSITORY, GITHUB_PULL_REQUEST_NUMBER, and GITHUB_RUN_ID. Updated scripts/github-action.sh to accept zero arguments (reading from env vars) or three positional arguments for backward compatibility.

All run: lines now only contain static shell variable references ($GITHUB_ACTION_PATH/scripts/...) with no ${{ }} template expressions.

