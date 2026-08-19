<!-- markdownlint-disable -->

# Hardening Report: lukka--get-cmake/v4.3.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lukka--get-cmake/v4.3.3** was hardened automatically. 3 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All `uses:` references across every workflow file use mutable tags or version strings instead of pinned 40-character commit SHAs, making the workflows vulnerable to supply-chain attacks if any referenced action is compromised or its tag is moved. Failing references include: actions/checkout@v5, actions/download-artifact@v8, actions/setup-node@v6, actions/upload-artifact@v7, peter-evans/create-pull-request@v7.

Locations:

- `.github/workflows/auto-release.yml:28`
- `.github/workflows/build-test-tmpl.yml:18`
- `.github/workflows/build-test-tmpl.yml:21`
- `.github/workflows/build-test-tmpl.yml:24`
- `.github/workflows/build-test-tmpl.yml:55`
- `.github/workflows/build-test-tmpl.yml:77`
- `.github/workflows/functional-tests-tmpl.yml:27`
- `.github/workflows/functional-tests-tmpl.yml:30`
- `.github/workflows/sync-latest-and-tag.yml:27`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions (sub-rule a), allowing template substitution before the shell parses the command. In sync-latest-and-tag.yml, the user-supplied `inputs.tag_name` is interpolated directly into shell commands: `TAG_NAME="${{ inputs.tag_name }}"` (Validate tag name step) and again in the Create and push tag step, and in the Summary step. In auto-release.yml, `${{ steps.check_version.outputs.CMAKE_VERSION }}` and `${{ github.repository }}` are interpolated directly into run: shell commands. In functional-tests-tmpl.yml, `${{ fromJSON(matrix.versions).cmake }}` and `${{ fromJSON(matrix.versions).ninja }}` are interpolated directly into run: shell commands (CMAKE_REQUESTED_VER and NINJA_REQUESTED_VER assignments and case statements).

Locations:

- `.github/workflows/sync-latest-and-tag.yml:42`
- `.github/workflows/sync-latest-and-tag.yml:68`
- `.github/workflows/sync-latest-and-tag.yml:87`
- `.github/workflows/sync-latest-and-tag.yml:100`
- `.github/workflows/auto-release.yml:56`
- `.github/workflows/auto-release.yml:65`
- `.github/workflows/auto-release.yml:68`
- `.github/workflows/auto-release.yml:77`
- `.github/workflows/auto-release.yml:88`
- `.github/workflows/functional-tests-tmpl.yml:47`
- `.github/workflows/functional-tests-tmpl.yml:72`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` block on any of their jobs. Without explicit permissions, workflows inherit the default repository token permissions (which may be broad). Affected files: build-test.yml (jobs: generate_catalog_build_and_test, build_and_test, build_and_test_arm, test_user_provided_version, test_user_provided_version_arm), build-test-tmpl.yml (job: build_and_test), and functional-tests-tmpl.yml (job: test_user_provided_versions).

Locations:

- `.github/workflows/build-test.yml:1`
- `.github/workflows/build-test-tmpl.yml:1`
- `.github/workflows/functional-tests-tmpl.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three finding types across 5 workflow files:

1. **unpinned-uses**: Pinned all 5 action references to full 40-char SHAs with tag comments preserved: actions/checkout@v5→fbc6f39, actions/download-artifact@v8→3e5f45b, actions/setup-node@v6→249970, actions/upload-artifact@v7→043fb46, peter-evans/create-pull-request@v7→22a9089.

2. **script-injection**: Moved all ${{ }} expressions out of run: blocks into env: blocks in:
   - sync-latest-and-tag.yml: inputs.tag_name→INPUT_TAG_NAME, github.repository→GH_REPOSITORY (3 steps)
   - auto-release.yml: steps.check_version.outputs.CMAKE_VERSION→CMAKE_VERSION, github.repository→GH_REPOSITORY, SHOULD_RELEASE→SHOULD_RELEASE (2 steps)
   - functional-tests-tmpl.yml: fromJSON(matrix.versions).cmake→CMAKE_REQUESTED_VER, fromJSON(matrix.versions).ninja→NINJA_REQUESTED_VER (2 steps)

3. **missing-permissions**: Added permissions blocks to:
   - build-test.yml: top-level `permissions: {}` (only calls reusable workflows)
   - build-test-tmpl.yml: job-level `contents: write, pull-requests: write` (creates PRs and commits)
   - functional-tests-tmpl.yml: job-level `contents: read` (read-only checkout)

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in `.github/workflows/functional-tests-tmpl.yml`:
1. Line 52: Changed `case ${CMAKE_REQUESTED_VER} in` to `case "${CMAKE_REQUESTED_VER}" in`
2. Line 80: Changed `case ${NINJA_REQUESTED_VER} in` to `case "${NINJA_REQUESTED_VER}" in`

Both variables are set from workflow-controllable matrix values (`${{ fromJSON(matrix.versions).cmake }}` and `${{ fromJSON(matrix.versions).ninja }}`). Quoting the expansions in the `case` statements prevents shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) in those values from being interpreted by the shell, eliminating the command injection risk.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two script injection vulnerabilities in .github/workflows/functional-tests-tmpl.yml (lines 75 and 99). The EXPECTED_CMAKE_VER and EXPECTED_NINJA_VER variables (derived from attacker-controllable matrix inputs) were used unquoted in bash [[ =~ ]] regex expressions, allowing injection of regex metacharacters. Fixed by replacing `=~ .*${EXPECTED_CMAKE_VER}.*` with `== *"${EXPECTED_CMAKE_VER}"*` and similarly for NINJA. Using bash [[ == ]] with a quoted variable in a glob pattern treats the variable value as a literal string, preventing any regex/glob injection.

