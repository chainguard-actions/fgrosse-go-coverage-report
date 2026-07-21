<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.3.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Download go-coverage-report' step directly interpolates ${{ inputs.version }} and ${{ inputs.sha256sum }} inside the run: shell command string. YAML template substitution occurs before the shell processes the string, so an attacker-controlled input value containing shell metacharacters (e.g. `; malicious-cmd`) can achieve command injection. The values should be passed via env: variables and referenced as quoted shell variables instead.

Locations:

- `action.yml:101`

### script-injection (severity: high)

Sub-rule (a): The 'Code coverage report' step directly interpolates ${{ github.repository }}, ${{ github.event.pull_request.number }}, and ${{ github.run_id }} inside the run: shell command string. Any ${{ ... }} expression interpolated directly into a run: block is a script-injection risk because YAML template substitution happens before the shell parses the string. These values should be passed via env: variables and referenced as quoted shell variables (e.g. "$GH_REPO") instead of being inlined as ${{ github.repository }}.

Locations:

- `action.yml:110`

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

Fixed all 4 script injection findings in action.yml:

1. 'Download go-coverage-report' step: Moved ${{ inputs.version }} and ${{ inputs.sha256sum }} from the run: command line into the env: block as VERSION and SHA256SUM. Updated scripts/download-cli-tool.sh to read VERSION and SHA256SUM from environment variables instead of positional arguments (with backwards compatibility).

2. 'Code coverage report' step: Moved ${{ github.repository }}, ${{ github.event.pull_request.number }}, and ${{ github.run_id }} from the run: command line into the env: block as GITHUB_REPOSITORY, GITHUB_PULL_REQUEST_NUMBER, and GITHUB_RUN_ID. Updated scripts/github-action.sh to read these values from environment variables instead of positional arguments (with backwards compatibility).

No ${{ }} expressions remain in any run: block — all template expressions are safely in env: blocks.

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 5 findings across 3 workflow files:

1. **script-injection** (ci.yml): Moved all 6 `${{ steps.coverage.outputs.* }}` expressions in the 'Print coverage outputs' step into an `env:` block (TOTAL_COVERAGE, COVERAGE_DELTA, COVERAGE_TREND, TOTAL_STATEMENTS, COVERED_STATEMENTS, MISSED_STATEMENTS). Shell script now references plain env vars.

2. **unpinned-uses** (ci.yml): Pinned `actions/checkout@v7` → `@3d3c42e5aac5ba805825da76410c181273ba90b1`, `actions/cache@v6` → `@55cc8345863c7cc4c66a329aec7e433d2d1c52a9`, `actions/upload-artifact@v7` → `@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a`.

3. **unpinned-uses** (codeql.yml): Pinned `actions/checkout@v7` → `@3d3c42e5aac5ba805825da76410c181273ba90b1`, `github/codeql-action/init@v4` → `@7188fc363630916deb702c7fdcf4e481b751f97a`, `github/codeql-action/analyze@v4` → `@7188fc363630916deb702c7fdcf4e481b751f97a`.

4. **unpinned-uses** (lint.yml): Pinned all 3 occurrences of `actions/checkout@v7` → `@3d3c42e5aac5ba805825da76410c181273ba90b1`.

5. **missing-permissions** (ci.yml): Added top-level `permissions: {}` to deny all by default, added `permissions: contents: read` to the `unit_tests` job, and added `contents: read` to the existing `code_coverage` job permissions block.

