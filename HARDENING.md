<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.5.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.5.0** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download go-coverage-report' step puts expressions straight into the run: script: `run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"`. The values are substituted into the script text before bash parses it, so a value containing a double quote or $(...) can inject commands. Fix: pass them through env (e.g. VERSION, SHA256SUM) and reference them as "$VERSION" and ${SHA256SUM:+"$SHA256SUM"}.

Locations:

- `action.yml:146`

### script-injection (severity: high)

Rule (a): the 'Code coverage report' step puts expressions straight into the run: script: `run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}"`. Any expression substituted into a run: script is a finding. Fix: route them through env (GITHUB_REPOSITORY, GITHUB_RUN_ID and an env var for the PR number) and reference them as double-quoted shell variables.

Locations:

- `action.yml:165`

### github-env-injection (severity: high)

Pattern (f), fixed heredoc delimiter: scripts/github-action.sh (run by the composite step) writes the multi-line coverage report to $GITHUB_OUTPUT with the fixed delimiter END_OF_COVERAGE_REPORT: `echo "coverage_report<<END_OF_COVERAGE_REPORT"; cat "$COVERAGE_COMMENT_PATH"; echo "END_OF_COVERAGE_REPORT"`. The report contains workflow- and attacker-influenced content: changed file and package names, TARGET_BRANCH and COVERAGE_ARTIFACT_NAME in the caution text. If a line in it equals the delimiter, the block ends early and later lines are read as extra outputs. Fix: build the delimiter from /dev/urandom on every run. The step also appends a metrics file produced by the tool to $GITHUB_OUTPUT (`cat "$METRICS_OUTPUT" >> "$GITHUB_OUTPUT"`), with no sanitization.

Locations:

- `scripts/github-action.sh`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:146`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha256sum }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:146`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index e7d6b3b..4dd828d 100644
--- a/action.yml
+++ b/action.yml
@@ -4,27 +4,22 @@ author: "Friedrich Große"
 branding:
   icon: 'umbrella'
   color: 'blue'
-
 inputs:
   version:
     description: 'The exact version tag of the go-coverage-report tool to use.'
     required: true
     default: "v1.5.0"
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
     description: |
       The Go import path of the tested repository to add as a prefix to all paths of the
@@ -35,25 +30,21 @@ inputs:
       ("github.com/owner/repo/v2"), vanity import paths or modules in a subdirectory.
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
@@ -62,12 +53,10 @@ inputs:
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
@@ -75,7 +64,6 @@ inputs:
       For other event types (e.g. push, schedule), set this explicitly.
     default: ${{ github.base_ref }}
     required: false
-
   baseline-run-id:
     description: |
       The ID of the GitHub Actions workflow run whose coverage artifact should be used as
@@ -83,7 +71,6 @@ inputs:
       for scripting, e.g. to feed the baseline run ID from another workflow into this action.
     default: ""
     required: false
-
   baseline-sha:
     description: |
       The commit whose coverage should be used as baseline. The action uses the latest
@@ -94,7 +81,6 @@ inputs:
       Set to "" to disable matching by commit and always use the latest run on the target branch.
     default: ${{ github.event.pull_request.base.sha }}
     required: false
-
   baseline-search-depth:
     description: |
       The number of commits in the first-parent history of baseline-sha (including itself) that
@@ -103,7 +89,6 @@ inputs:
       100 commits, so if it contains merged branches, fewer commits may be searched.
     default: "30"
     required: false
-
   baseline-max-downloads:
     description: |
       Stop searching the history of baseline-sha after this many coverage artifacts could not be
@@ -111,7 +96,6 @@ inputs:
       run on the target branch is still tried afterwards.
     default: "5"
     required: false
-
 outputs:
   coverage_report:
     description: 'The generated coverage report in Markdown format.'
@@ -134,22 +118,38 @@ outputs:
   missed_statements:
     description: 'Number of missed (uncovered) statements in the new coverage profile.'
     value: ${{ steps.coverage.outputs.missed_statements }}
-
 runs:
   using: "composite"
-
   steps:
     - name: Download go-coverage-report
       she
... (truncated)
```

Locations:

- `action.yml`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed scripts/download-cli-tool.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/scripts/download-cli-tool.sh b/scripts/download-cli-tool.sh
index 59c5818..1de8ae9 100755
--- a/scripts/download-cli-tool.sh
+++ b/scripts/download-cli-tool.sh
@@ -23,23 +23,18 @@ You can use the following environment variables to configure the script:
 - DRY_RUN: Do not actually move the binary to /usr/bin (default: false)
 "
 
-if [[ $# == 0 ]]; then
-  echo -e "Error: script requires at least one argument\n"
-  echo "$USAGE"
-  exit 1
-fi
-
-VERSION=$1
+VERSION=${VERSION:-${1:-}}
 RUNNER_OS=${RUNNER_OS:-Linux}
 RUNNER_ARCH=${RUNNER_ARCH:-X64}
 
-if [[ -z ${VERSION+x} ]]; then
-    echo "Missing version argument"
+if [[ -z "$VERSION" ]]; then
+    echo -e "Error: VERSION environment variable (or first positional argument) is required\n"
+    echo "$USAGE"
     exit 1
 fi
 
 if [[ $# == 2 ]]; then
-  SHA256SUM=$2
+  SHA256SUM=${SHA256SUM:-$2}
 fi
 
 sudo_or_dry_run="sudo"

```

Locations:

- `scripts/download-cli-tool.sh`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed scripts/github-action.sh without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/scripts/github-action.sh b/scripts/github-action.sh
index 77f7990..a823e6a 100755
--- a/scripts/github-action.sh
+++ b/scripts/github-action.sh
@@ -43,15 +43,21 @@ You can use the following environment variables to configure the script:
 - SKIP_COMMENT: Skip creating or updating the pull request comment (default: false)
 "
 
-if [[ $# != 3 ]]; then
-  echo -e "Error: script requires exactly three arguments\n"
+if [[ $# == 3 ]]; then
+  GITHUB_REPOSITORY=${GITHUB_REPOSITORY:-$1}
+  GITHUB_PULL_REQUEST_NUMBER=${GITHUB_PULL_REQUEST_NUMBER:-$2}
+  GITHUB_RUN_ID=${GITHUB_RUN_ID:-$3}
+elif [[ $# != 0 ]]; then
+  echo -e "Error: script requires either zero arguments (using environment variables) or exactly three positional arguments\n"
   echo "$USAGE"
   exit 1
 fi
 
-GITHUB_REPOSITORY=$1
-GITHUB_PULL_REQUEST_NUMBER=$2
-GITHUB_RUN_ID=$3
+if [[ -z "${GITHUB_REPOSITORY:-}" ]]; then
+  echo -e "Error: GITHUB_REPOSITORY environment variable (or first positional argument) is required\n"
+  echo "$USAGE"
+  exit 1
+fi
 GITHUB_BASELINE_WORKFLOW=${GITHUB_BASELINE_WORKFLOW:-CI}
 TARGET_BRANCH=${TARGET_BRANCH:-main}
 EVENT_NAME=${EVENT_NAME:-push}

```

Locations:

- `scripts/github-action.sh`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

I fixed all findings. I haven't run the action or tested it.

action.yml:
- 'Download go-coverage-report' step: inputs.version and inputs.sha256sum now come in through env (INPUT_VERSION, INPUT_SHA256SUM) and are passed as "$INPUT_VERSION" "$INPUT_SHA256SUM". I passed the checksum as a plain quoted argument rather than ${SHA256SUM:+...}. Upstream always passed two arguments, so this keeps the script's argument count the same.
- 'Code coverage report' step: github.repository, the PR number and github.run_id now come in through env (ACTION_REPOSITORY, ACTION_PR_NUMBER, ACTION_RUN_ID) and are passed as quoted positional arguments. The script path is quoted too.

scripts/github-action.sh:
- The coverage_report delimiter written to $GITHUB_OUTPUT is now random on every run (END_OF_COVERAGE_REPORT_ plus 16 bytes from /dev/urandom). The report's own lines are written unchanged.
- The metrics file is no longer copied straight into $GITHUB_OUTPUT. Only lines for the six known metric keys, with values made of letters, digits, spaces and '.', '+', '-', get through, and carriage returns are removed. This matches everything the tool writes in WriteMetrics.

prior-fix: the previous harden changed both scripts to read their values from environment variables. I didn't re-apply that. The scripts still take positional arguments exactly as upstream wrote them, and action.yml now fills those arguments from env vars, so nothing is substituted into the script text any more. The script argument handling is unchanged from upstream.

