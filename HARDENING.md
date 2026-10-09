<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download go-coverage-report' step directly interpolates `${{ inputs.version }}` and `${{ inputs.sha256sum }}` inside the `run:` shell command string. These expressions are expanded by the GitHub Actions template engine before the shell ever sees them, allowing an attacker who controls these inputs to inject arbitrary shell commands. Offending line: `run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"`

Fix: move the values into `env:` variables and reference them as quoted shell variables, e.g.:
```yaml
env:
  VERSION: ${{ inputs.version }}
  SHA256SUM: ${{ inputs.sha256sum }}
run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "$VERSION" "$SHA256SUM"
```

Locations:

- `action.yml:105`

### script-injection (severity: high)

Sub-rule (a): The 'Code coverage report' step directly interpolates `${{ github.repository }}`, `${{ github.event.pull_request.number }}`, and `${{ github.run_id }}` inside the `run:` shell command string. These expressions are expanded by the GitHub Actions template engine before the shell sees them. `github.repository` and `github.event.pull_request.number` can be attacker-influenced (e.g. via a fork PR), enabling shell command injection. Offending line: `run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}"`

Fix: pass these values via `env:` variables (which are already partially present in the step's `env:` block) and reference them as quoted shell variables inside the script, e.g.:
```yaml
env:
  GITHUB_REPOSITORY: ${{ github.repository }}
  GITHUB_PULL_REQUEST_NUMBER: ${{ github.event.pull_request.number }}
  GITHUB_RUN_ID: ${{ github.run_id }}
run: $GITHUB_ACTION_PATH/scripts/github-action.sh
```

Locations:

- `action.yml:123`

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

Fixed all four script-injection findings in action.yml:

1. 'Download go-coverage-report' step: Moved `${{ inputs.version }}` and `${{ inputs.sha256sum }}` from the `run:` command string into the `env:` block as `VERSION` and `SHA256SUM`. The run command now uses `"$VERSION"` and `"$SHA256SUM"` as quoted shell variables.

2. 'Code coverage report' step: Moved `${{ github.repository }}`, `${{ github.event.pull_request.number }}`, and `${{ github.run_id }}` from the `run:` command string into the `env:` block as `GITHUB_REPOSITORY`, `GITHUB_PULL_REQUEST_NUMBER`, and `GITHUB_RUN_ID`. The script is now called without positional arguments.

3. Updated `scripts/download-cli-tool.sh` to read `VERSION` from the environment variable when no positional argument is provided, while maintaining backward compatibility with the positional argument form.

4. Updated `scripts/github-action.sh` to accept zero arguments (reading from environment variables) in addition to the original three-argument form, maintaining backward compatibility.

No `${{ }}` expressions remain in any `run:` blocks in action.yml.

