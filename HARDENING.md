<!-- markdownlint-disable -->

# Hardening Report: fgrosse--go-coverage-report/v1.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **fgrosse--go-coverage-report/v1.3.1** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): the 'Download go-coverage-report' step puts GitHub expressions straight into the run: script: `run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "${{ inputs.version }}" "${{ inputs.sha256sum }}"`. The runner substitutes inputs.version and inputs.sha256sum into the script text before bash parses it, so a value containing a double quote followed by shell metacharacters runs as a command. Fix: move these values into env: variables and pass them as quoted "$VAR" arguments.

Locations:

- `action.yml:105`

### script-injection (severity: high)

Rule (a): the 'Code coverage report' step puts GitHub expressions straight into the run: script: `run: $GITHUB_ACTION_PATH/scripts/github-action.sh "${{ github.repository }}" "${{ github.event.pull_request.number }}" "${{ github.run_id }}"`. Any ${{ }} expression inside run: is template-substituted into the script before bash parses it. Fix: pass these through env: variables (or the built-in GITHUB_REPOSITORY and GITHUB_RUN_ID variables) and use quoted shell expansions.

Locations:

- `action.yml:124`

### github-env-injection (severity: high)

Rule (f), multi-line write with a fixed heredoc delimiter: scripts/github-action.sh writes the coverage report to $GITHUB_OUTPUT using the hardcoded delimiter END_OF_COVERAGE_REPORT (`echo "coverage_report<<END_OF_COVERAGE_REPORT"; cat "$COVERAGE_COMMENT_PATH"; echo "END_OF_COVERAGE_REPORT"`). The report content comes from workflow-controlled data: changed file names from the PR, ROOT_PACKAGE, TRIM_PACKAGE and coverage artifacts. If any line of the report equals the delimiter, the block ends early and every later line becomes a new output entry. Fix: generate the delimiter from a secure random source on every run, e.g. EOF=$(openssl rand -hex 16) or /dev/urandom.

Locations:

- `scripts/github-action.sh:177`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.version }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:106`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.sha256sum }}" appears directly in run: block of step "Download go-coverage-report"; move to env: map

Locations:

- `action.yml:106`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index c095629..d010dea 100644
--- a/action.yml
+++ b/action.yml
@@ -4,52 +4,38 @@ author: "Friedrich Große"
 branding:
   icon: 'umbrella'
   color: 'blue'
-
 inputs:
   version:
     description: 'The exact version tag of the go-coverage-report tool to use.'
     required: true
     default: "v1.3.1"
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
     description: Exclude files matching the given regular expression from the report.
     required: false
-
   github-baseline-workflow-ref:
     description: |
       The ref of the GitHub actions Workflow that produces the baseline coverage.
@@ -58,12 +44,10 @@ inputs:
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
@@ -71,7 +55,6 @@ inputs:
       For other event types (e.g. push, schedule), set this explicitly.
     default: ${{ github.base_ref }}
     required: false
-
 outputs:
   coverage_report:
     description: 'The generated coverage report in Markdown format.'
@@ -94,22 +77,38 @@ outputs:
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
+      run: $GITHUB_ACTION_PATH/scripts/download-cli-tool.sh "$VERSION" "$SHA256SUM"
       env:
         RUNNER_OS: ${{ runner.os }}
         RUNNER_ARCH: ${{ runner.arch }}
-
+        VERSION: ${{ inputs.version }}
+        SHA256SUM: ${{ inputs.sha256sum }}
     - name: Determine changed files
       id: changed-files
       uses: tj-actions/changed-files@9426d40962ed5378910ee2e21d5f8c6fcbf2dd96 # v47.0.6
       with:
         write_output_files: true
         json: true
@@ -117,11 +116,10 @@ runs:
           **.go
         files_ignore: |
           vendor/**
-
     -
... (truncated)
```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection, prior-fix

**Notes:**

In action.yml, both run: lines no longer contain ${{ }} expressions. The download step takes inputs.version and inputs.sha256sum from the env: variables INPUT_VERSION and INPUT_SHA256SUM. The coverage step takes the repository from the existing GH_REPO variable, and the PR number and run ID from new PR_NUMBER and RUN_ID variables. Every argument and the script path are double-quoted, and the scripts still get the same 3-argument / 2-argument counts as upstream. In scripts/github-action.sh, the fixed END_OF_COVERAGE_REPORT heredoc delimiter is now random on every run (16 bytes read from /dev/urandom). The report itself is written unchanged. This re-applies the security part of the previous harden; that patch also reformatted action.yml, which I did not repeat. Nothing was run or tested.

