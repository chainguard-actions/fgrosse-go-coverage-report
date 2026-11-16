<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.4.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download go-coverage-report' step puts GitHub expressions straight into the run: script: `run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"`. The values are substituted into the script text before bash parses it, so a crafted input with `"` or `$(...)` can run commands. Fix: pass them through env: (e.g. VERSION, SHA256SUM) and refer to them as "$VERSION" "$SHA256SUM".

Locations:

- `action.yml:109`

### script-injection (severity: high)

Rule (a): the 'Code coverage report' step puts GitHub expressions straight into the run: script: `run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}"`. Fix: pass these through env: variables and refer to them in double quotes (e.g. "$GITHUB_REPOSITORY" "$PR_NUMBER" "$GITHUB_RUN_ID").

Locations:

- `action.yml:128`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:109`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha256sum }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:109`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 1fe0104..3066446 100644
--- a/action.yml
+++ b/action.yml
@@ -4,55 +4,41 @@ author: "Friedrich Große"
 branding:
   icon: 'umbrella'
   color: 'blue'
-
 inputs:
   version:
     description: 'The exact version tag of the go-coverage-report tool to use.'
     required: true
     default: "v1.4.0"
-
   sha256sum:
     description: 'Optional SHA256 checksum of the tarball when downloading the go-coverage-report binary.'
     required: false
-
   coverage-artifact-name:
     description: 'The name of the artifact containing the code coverage results.'
     required: true
     default: "code-coverage"
-
   coverage-file-name:
     description: 'The name of the file containing the code coverage results.'
     required: true
     default: "coverage.txt"
-
   root-package:
-    description: |
-      The Go import path of the tested repository to add as a prefix to all paths of the
-      changed files. This is useful to map the changed files (e.g., ["foo/my_file.go"]
-      to their coverage profile which uses the full package name to identify the files
-      (e.g., "github.com/fgrosse/example/foo/my_file.go"). Note that currently, 
-      packages with a different name than their directory are not supported.
+    description: "The Go import path of the tested repository to add as a prefix to all paths of the\nchanged files. This is useful to map the changed files (e.g., [\"foo/my_file.go\"]\nto their coverage profile which uses the full package name to identify the files\n(e.g., \"github.com/fgrosse/example/foo/my_file.go\"). Note that currently, \npackages with a different name than their directory are not supported.\n"
     required: false
     default: "github.com/${{ github.repository }}"
-
   skip-comment:
     description: |
       Skip creating or updating the pull request comment. This may be useful when you want
       to generate the coverage report and modify it in your own scripts.
     required: false
     default: 'false'
-
   trim:
     description: Trim a prefix in the "Impacted Packages" column of the markdown report.
     required: false
-
   exclude:
     description: |
       Exclude files matching the given regular expression from the report. The expression
       is applied to the file names in the coverage profiles as well as to the changed files
       (after the root-package prefix was added).
     required: false
-
   github-baseline-workflow-ref:
     description: |
       The ref of the GitHub actions Workflow that produces the baseline coverage.
@@ -61,12 +47,10 @@ inputs:
       You can aso just pass the name of the Workflow file directly (e.g. "my-workflow.yml").
     default: ${{ github.workflow_ref }}
     required: false
-
   event-name:
     description: "The event that triggered the workflow used to filter baseline runs."
     default: "push"
     required: false
-
   target-branch:
     description: |
       The base branch to compare the coverage results against.
@@ -74,7 +58,6 @@ inputs:
       For other event types (e.g. push, schedule), set this explicitly.
     default: ${{ github.base_ref }}
     required: false
-
 outputs:
   coverage_report:
     description: 'The generated coverage report in Markdown format.'
@@ -97,22 +80,38 @@ outputs:
   missed_statements:
     description: 'Number of missed (uncovered) statements in the new coverage profile.'
     value: ${{ steps.coverage.outputs.missed_statements }}
-
 runs:
   using: "composite"
-
   steps:
     - name: Download go-coverage-report
       shell: bash
       id: download
-      run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"
+      run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh
       env:
         RUNNER_OS: ${{ runner.os }}
         RUNNER_ARCH: ${{ runner.arch }}
-
+        INPUT_VERSION: ${{ inputs.version }}
+        INPUT_SHA256SUM: ${{ inputs.sha256sum }}
     - name: Determine changed files
       id: changed-files
       uses: tj-actions/changed-files@9426d40962ed5378910ee2e21d5f8c6fcbf2dd96 # v47.0.6
       with
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed scripts/download-cli-tool.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/scripts/download-cli-tool.sh b/scripts/download-cli-tool.sh
index 59c5818..c6b1d55 100755
--- a/scripts/download-cli-tool.sh
+++ b/scripts/download-cli-tool.sh
@@ -23,24 +23,16 @@ You can use the following environment variables to configure the script:
 - DRY_RUN: Do not actually move the binary to /usr/bin (default: false)
 "
 
-if [[ $# == 0 ]]; then
-  echo -e "Error: script requires at least one argument\n"
-  echo "$USAGE"
-  exit 1
-fi
-
-VERSION=$1
+VERSION=${INPUT_VERSION:-}
 RUNNER_OS=${RUNNER_OS:-Linux}
 RUNNER_ARCH=${RUNNER_ARCH:-X64}
 
-if [[ -z ${VERSION+x} ]]; then
-    echo "Missing version argument"
+if [[ -z "$VERSION" ]]; then
+    echo "Missing version (INPUT_VERSION environment variable)"
     exit 1
 fi
 
-if [[ $# == 2 ]]; then
-  SHA256SUM=$2
-fi
+SHA256SUM=${INPUT_SHA256SUM:-}
 
 sudo_or_dry_run="sudo"
 if [[ "$DRY_RUN" == "true" ]]; then

```

Locations:

- `scripts/download-cli-tool.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed scripts/github-action.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/scripts/github-action.sh b/scripts/github-action.sh
index 13be554..a28f0ef 100755
--- a/scripts/github-action.sh
+++ b/scripts/github-action.sh
@@ -37,15 +37,9 @@ You can use the following environment variables to configure the script:
 - SKIP_COMMENT: Skip creating or updating the pull request comment (default: false)
 "
 
-if [[ $# != 3 ]]; then
-  echo -e "Error: script requires exactly three arguments\n"
-  echo "$USAGE"
-  exit 1
-fi
-
-GITHUB_REPOSITORY=$1
-GITHUB_PULL_REQUEST_NUMBER=$2
-GITHUB_RUN_ID=$3
+GITHUB_REPOSITORY=${INPUT_GITHUB_REPOSITORY:-}
+GITHUB_PULL_REQUEST_NUMBER=${INPUT_GITHUB_PULL_REQUEST_NUMBER:-}
+GITHUB_RUN_ID=${INPUT_GITHUB_RUN_ID:-}
 GITHUB_BASELINE_WORKFLOW=${GITHUB_BASELINE_WORKFLOW:-CI}
 TARGET_BRANCH=${TARGET_BRANCH:-main}
 EVENT_NAME=${EVENT_NAME:-push}

```

Locations:

- `scripts/github-action.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, prior-fix

**Notes:**

I moved every ${{ }} expression out of the two run: scripts in action.yml and into each step's env: block. Download step: inputs.version is now INPUT_VERSION and inputs.sha256sum is INPUT_SHA256SUM. Coverage step: github.repository, github.event.pull_request.number and github.run_id are now INPUT_GITHUB_REPOSITORY, INPUT_GITHUB_PULL_REQUEST_NUMBER and INPUT_GITHUB_RUN_ID. The scripts still get these values as the same positional arguments, double-quoted, so they receive exactly what upstream passed them, including an empty sha256sum argument when that input is unset. That fixes the injection without the script rewrites the previous harden made. I left scripts/download-cli-tool.sh and scripts/github-action.sh unchanged on purpose, because the inputs no longer reach the shell as script text. The previous harden's other changes to action.yml were removed blank lines and a reflowed description, not security fixes, so I didn't redo them. Nothing was run or tested.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In scripts/github-action.sh, the fixed END_OF_COVERAGE_REPORT heredoc delimiter used when writing coverage_report to $GITHUB_OUTPUT is now a random one generated on every run: EOF_DELIM="EOF_$(od -An -N16 -tx1 /dev/urandom | tr -d ' \n')". The multi-line report is written unchanged and its newlines are kept. I didn't need to change action.yml: it already passes all ${{ }} values through env:, and the line flagged there only runs the script.

