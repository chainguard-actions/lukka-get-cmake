<!-- markdownlint-disable -->

# Hardening Report: lukka--get-cmake/v4.3.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lukka--get-cmake/v4.3.1** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple ${{ }} expressions are interpolated directly inside run: shell scripts. In 'Create and push version tag' step: `TAG_NAME="v${{ steps.check_version.outputs.CMAKE_VERSION }}"`, `/repos/${{ github.repository }}/git/tags`, and `-f message="Release $TAG_NAME - CMake ${{ steps.check_version.outputs.CMAKE_VERSION }}"`. In 'Create release summary' step: `if [[ "${{ steps.check_version.outputs.SHOULD_RELEASE }}" == "true" ]]` and multiple `${{ steps.check_version.outputs.CMAKE_VERSION }}` in echo commands. These expressions are substituted by the YAML template engine before the shell ever sees them, enabling command injection.

Locations:

- `.github/workflows/auto-release.yml:68`
- `.github/workflows/auto-release.yml:74`
- `.github/workflows/auto-release.yml:76`
- `.github/workflows/auto-release.yml:87`
- `.github/workflows/auto-release.yml:99`

### script-injection (severity: high)

Sub-rule (a): ${{ inputs.tag_name }} (attacker-controllable via workflow_dispatch) and ${{ github.repository }} are interpolated directly inside run: shell scripts. In 'Validate tag name' step: `TAG_NAME="${{ inputs.tag_name }}"`. In 'Create and push tag' step: `TAG_NAME="${{ inputs.tag_name }}"` and `/repos/${{ github.repository }}/git/tags`. In 'Summary' step: `echo "- ✓ Tag \`${{ inputs.tag_name }}\` created"` and `echo "- \`@${{ inputs.tag_name }}\` - for this specific version"`. An attacker with workflow_dispatch access can inject arbitrary shell commands via the tag_name input.

Locations:

- `.github/workflows/sync-latest-and-tag.yml:38`
- `.github/workflows/sync-latest-and-tag.yml:68`
- `.github/workflows/sync-latest-and-tag.yml:74`
- `.github/workflows/sync-latest-and-tag.yml:108`
- `.github/workflows/sync-latest-and-tag.yml:111`

### script-injection (severity: high)

Sub-rule (a) and (b): In 'CMake version check' step, `CMAKE_REQUESTED_VER="${{ fromJSON(matrix.versions).cmake }}"` interpolates a matrix expression directly into the shell script (sub-rule a). The variable is then used unquoted in `case ${CMAKE_REQUESTED_VER} in` (sub-rule b), allowing shell metacharacter injection. Similarly in 'ninja version check' step: `NINJA_REQUESTED_VER="${{ fromJSON(matrix.versions).ninja }}"` (sub-rule a) and unquoted `case ${NINJA_REQUESTED_VER} in` (sub-rule b). The matrix.versions values are workflow-caller-controlled.

Locations:

- `.github/workflows/functional-tests-tmpl.yml:43`
- `.github/workflows/functional-tests-tmpl.yml:44`
- `.github/workflows/functional-tests-tmpl.yml:68`
- `.github/workflows/functional-tests-tmpl.yml:69`

### unpinned-uses (severity: high)

The following uses: references are pinned to mutable tags rather than full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved: `actions/checkout@v5` (line 28).

Locations:

- `.github/workflows/auto-release.yml:28`

### unpinned-uses (severity: high)

The following uses: references are pinned to mutable tags rather than full 40-character commit SHAs: `actions/checkout@v5` (line 25), `actions/download-artifact@v8` (line 28), `actions/setup-node@v6` (line 31), `actions/upload-artifact@v7` (line 54), `peter-evans/create-pull-request@v7` (line 72).

Locations:

- `.github/workflows/build-test-tmpl.yml:25`
- `.github/workflows/build-test-tmpl.yml:28`
- `.github/workflows/build-test-tmpl.yml:31`
- `.github/workflows/build-test-tmpl.yml:54`
- `.github/workflows/build-test-tmpl.yml:72`

### unpinned-uses (severity: high)

The following uses: references are pinned to mutable tags rather than full 40-character commit SHAs: `actions/checkout@v5` (line 30), `actions/setup-node@v6` (line 33).

Locations:

- `.github/workflows/functional-tests-tmpl.yml:30`
- `.github/workflows/functional-tests-tmpl.yml:33`

### unpinned-uses (severity: high)

The following uses: reference is pinned to a mutable tag rather than a full 40-character commit SHA: `actions/checkout@v5` (line 26).

Locations:

- `.github/workflows/sync-latest-and-tag.yml:26`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the job `generate_catalog_build_and_test`, `build_and_test`, `build_and_test_arm`, `test_user_provided_version`, and `test_user_provided_version_arm` have no job-level `permissions:` key. This means the GITHUB_TOKEN is granted default (potentially broad) permissions.

Locations:

- `.github/workflows/build-test.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the job `build_and_test` has no job-level `permissions:` key. This means the GITHUB_TOKEN is granted default (potentially broad) permissions.

Locations:

- `.github/workflows/build-test-tmpl.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and the job `test_user_provided_versions` has no job-level `permissions:` key. This means the GITHUB_TOKEN is granted default (potentially broad) permissions.

Locations:

- `.github/workflows/functional-tests-tmpl.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 10 findings across 5 workflow files:

1. auto-release.yml: Pinned actions/checkout@v5 to SHA fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09. Fixed script injection in 'Create and push version tag' step (CMAKE_VERSION, GH_REPOSITORY moved to env:) and 'Create release summary' step (SHOULD_RELEASE, CMAKE_VERSION moved to env:).

2. build-test-tmpl.yml: Added job-level permissions (contents: write, pull-requests: write). Pinned all 5 actions: actions/checkout@v5→fbc6f39, actions/download-artifact@v8→3e5f45b, actions/setup-node@v6→249970729, actions/upload-artifact@v7→043fb46, peter-evans/create-pull-request@v7→22a9089.

3. functional-tests-tmpl.yml: Added job-level permissions (contents: read). Pinned actions/checkout@v5→fbc6f39 and actions/setup-node@v6→249970729. Fixed script injection in CMake and ninja version check steps by moving fromJSON(matrix.versions) expressions to env: blocks and quoting the case statement variables.

4. sync-latest-and-tag.yml: Pinned actions/checkout@v5→fbc6f39. Fixed script injection in 'Validate tag name' (TAG_NAME to env:), 'Create and push tag' (TAG_NAME, GH_REPOSITORY to env:), and 'Summary' (TAG_NAME to env:) steps.

5. build-test.yml: Added top-level `permissions: {}` to restrict default GITHUB_TOKEN permissions.

