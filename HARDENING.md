<!-- markdownlint-disable -->

# Hardening Report: lukka--get-cmake/v4.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lukka--get-cmake/v4.2.3** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): ${{ }} expressions are interpolated directly inside run: shell command strings. In auto-release.yml, `${{ steps.check_version.outputs.CMAKE_VERSION }}` and `${{ steps.check_version.outputs.SHOULD_RELEASE }}` are embedded directly in run: blocks (e.g., `TAG_NAME="v${{ steps.check_version.outputs.CMAKE_VERSION }}"`; `if [[ "${{ steps.check_version.outputs.SHOULD_RELEASE }}" == "true" ]]`). These step outputs flow through YAML template substitution before the shell sees them, enabling shell metacharacter injection.

Locations:

- `.github/workflows/auto-release.yml:62`
- `.github/workflows/auto-release.yml:71`
- `.github/workflows/auto-release.yml:80`

### script-injection (severity: high)

Rule (a): `${{ inputs.tag_name }}` (a workflow_dispatch user-controlled input) is interpolated directly inside run: shell command strings in sync-latest-and-tag.yml. For example: `TAG_NAME="${{ inputs.tag_name }}"` appears in both the 'Validate tag name' and 'Create and push tag' steps. An attacker supplying a crafted tag_name value could inject shell commands.

Locations:

- `.github/workflows/sync-latest-and-tag.yml:38`
- `.github/workflows/sync-latest-and-tag.yml:64`
- `.github/workflows/sync-latest-and-tag.yml:74`

### script-injection (severity: high)

Rule (a): `${{ fromJSON(matrix.versions).cmake }}` and `${{ fromJSON(matrix.versions).ninja }}` are interpolated directly inside run: shell command strings in functional-tests-tmpl.yml. For example: `CMAKE_REQUESTED_VER="${{ fromJSON(matrix.versions).cmake }}"` and `NINJA_REQUESTED_VER="${{ fromJSON(matrix.versions).ninja }}"`. Matrix values flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/functional-tests-tmpl.yml:40`
- `.github/workflows/functional-tests-tmpl.yml:74`

### unpinned-uses (severity: high)

Multiple workflow files reference external actions using mutable tag refs instead of immutable 40-character SHA commit hashes. Failing references include: actions/checkout@v5, actions/download-artifact@v6, actions/setup-node@v6, actions/upload-artifact@v5, peter-evans/create-pull-request@v7. Any of these tags could be moved to point to malicious code without notice.

Locations:

- `.github/workflows/auto-release.yml:28`
- `.github/workflows/build-test-tmpl.yml:19`
- `.github/workflows/build-test-tmpl.yml:22`
- `.github/workflows/build-test-tmpl.yml:25`
- `.github/workflows/build-test-tmpl.yml:55`
- `.github/workflows/build-test-tmpl.yml:93`
- `.github/workflows/functional-tests-tmpl.yml:26`
- `.github/workflows/functional-tests-tmpl.yml:29`
- `.github/workflows/sync-latest-and-tag.yml:26`

### missing-permissions (severity: medium)

build-test-tmpl.yml has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`build_and_test`). This means the job runs with the default (potentially broad) GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/build-test-tmpl.yml:1`

### missing-permissions (severity: medium)

build-test.yml has no top-level `permissions:` key and none of its jobs (`generate_catalog_build_and_test`, `build_and_test`, `build_and_test_arm`, `test_user_provided_version`, `test_user_provided_version_arm`) have a `permissions:` key. All jobs run with default GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/build-test.yml:1`

### missing-permissions (severity: medium)

functional-tests-tmpl.yml has no top-level `permissions:` key and no job-level `permissions:` key on its only job (`test_user_provided_versions`). The job runs with default GITHUB_TOKEN permissions.

Locations:

- `.github/workflows/functional-tests-tmpl.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 7 findings across 5 workflow files:

1. auto-release.yml: Pinned actions/checkout@v5 to SHA 93cb6efe...; moved ${{ steps.check_version.outputs.CMAKE_VERSION }} and ${{ steps.check_version.outputs.SHOULD_RELEASE }} from run: blocks into env: blocks.

2. sync-latest-and-tag.yml: Pinned actions/checkout@v5 to SHA 93cb6efe...; moved ${{ inputs.tag_name }} from all run: blocks into env: blocks.

3. functional-tests-tmpl.yml: Pinned actions/checkout@v5 (93cb6efe) and actions/setup-node@v6 (24997072); added top-level permissions: contents: read; moved ${{ fromJSON(matrix.versions).cmake }} and ${{ fromJSON(matrix.versions).ninja }} from run: blocks into env: blocks.

4. build-test-tmpl.yml: Pinned actions/checkout@v5 (93cb6efe), actions/download-artifact@v6 (018cc2cf), actions/setup-node@v6 (24997072), actions/upload-artifact@v5 (330a01c4), peter-evans/create-pull-request@v7 (22a90890); added top-level permissions: contents: read and job-level permissions: contents: write, pull-requests: write.

5. build-test.yml: Added top-level permissions: contents: read.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in case statements in `.github/workflows/functional-tests-tmpl.yml`. Changed `case ${CMAKE_REQUESTED_VER} in` to `case "${CMAKE_REQUESTED_VER}" in` (line 50) and `case ${NINJA_REQUESTED_VER} in` to `case "${NINJA_REQUESTED_VER}" in` (line 86). These variables are sourced from matrix context values and quoting them prevents shell metacharacter injection.

