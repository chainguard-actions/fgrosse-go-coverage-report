<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **fgrosse--go-coverage-report/v1.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Download go-coverage-report' step directly interpolates `${{ inputs.version }}` and `${{ inputs.sha256sum }}` into the `run:` shell command string. These are caller-controlled inputs that flow through YAML template substitution before the shell sees them, enabling command injection if an attacker can supply a crafted value (e.g. a version string containing shell metacharacters). The offending line is: `run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"`

Fix: Move the values into `env:` variables and reference them as quoted shell variables inside the script, e.g.:
```yaml
env:
  VERSION: ${{ inputs.version }}
  SHA256SUM: ${{ inputs.sha256sum }}
run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "$VERSION" "$SHA256SUM"
```

Locations:

- `action.yml:97`

### script-injection (severity: high)

Rule (a) violation: The 'Code coverage report' step directly interpolates `${{ github.repository }}`, `${{ github.event.pull_request.number }}`, and `${{ github.run_id }}` into the `run:` shell command string. Even though `github.run_id` is GitHub-controlled, all three values flow through YAML template substitution before the shell processes them, and `github.repository` / `github.event.pull_request.number` can be attacker-influenced (e.g. via a fork with a crafted repository name or PR number). The offending line is: `run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}"`

Fix: Move the values into `env:` variables and reference them as quoted shell variables:
```yaml
env:
  GITHUB_REPOSITORY: ${{ github.repository }}
  GITHUB_PULL_REQUEST_NUMBER: ${{ github.event.pull_request.number }}
  GITHUB_RUN_ID: ${{ github.run_id }}
run: $GITHUB_ACTION_PATH/scripts/github-action.sh "$GITHUB_REPOSITORY" "$GITHUB_PULL_REQUEST_NUMBER" "$GITHUB_RUN_ID"
```

Locations:

- `action.yml:117`

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

Fixed two script injection vulnerabilities in action.yml:
1. 'Download go-coverage-report' step: moved `${{ inputs.version }}` and `${{ inputs.sha256sum }}` into env: block as VERSION and SHA256SUM; updated run: to use `"$VERSION"` and `${SHA256SUM:+"$SHA256SUM"}` (conditional expansion for optional sha256sum).
2. 'Code coverage report' step: moved `${{ github.repository }}`, `${{ github.event.pull_request.number }}`, and `${{ github.run_id }}` into env: block as GITHUB_REPOSITORY, GITHUB_PULL_REQUEST_NUMBER, and GITHUB_RUN_ID; updated run: to reference these shell variables, using conditional expansion for the optional PR number.

