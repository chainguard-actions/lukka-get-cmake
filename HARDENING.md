<!-- markdownlint-disable -->

# Hardening Report: lukka--get-cmake/v4.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lukka--get-cmake/v4.4.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple run: blocks in sync-latest-and-tag.yml directly interpolate ${{ inputs.tag_name }} and ${{ github.repository }} into shell commands without routing through env: variables. This allows a workflow_dispatch caller to inject arbitrary shell commands. Affected steps: 'Validate tag name' (TAG_NAME="${{ inputs.tag_name }}"), 'Create and push tag' (TAG_NAME="${{ inputs.tag_name }}" and /repos/${{ github.repository }}/git/tags), and 'Summary' (echo "- ✓ Tag \`${{ inputs.tag_name }}\`" and echo "- \`@${{ inputs.tag_name }}\`").

Locations:

- `.github/workflows/sync-latest-and-tag.yml:39`
- `.github/workflows/sync-latest-and-tag.yml:57`
- `.github/workflows/sync-latest-and-tag.yml:68`
- `.github/workflows/sync-latest-and-tag.yml:83`
- `.github/workflows/sync-latest-and-tag.yml:93`
- `.github/workflows/sync-latest-and-tag.yml:96`

### script-injection (severity: high)

Multiple run: blocks in auto-release.yml directly interpolate ${{ steps.check_version.outputs.CMAKE_VERSION }}, ${{ github.repository }}, and ${{ steps.check_version.outputs.SHOULD_RELEASE }} into shell commands. Affected steps: 'Create and push version tag' (TAG_NAME="v${{ steps.check_version.outputs.CMAKE_VERSION }}", /repos/${{ github.repository }}/git/tags, -f message="Release $TAG_NAME - CMake ${{ steps.check_version.outputs.CMAKE_VERSION }}", /repos/${{ github.repository }}/git/refs) and 'Create release summary' (if [[ "${{ steps.check_version.outputs.SHOULD_RELEASE }}" == "true" ]], echo "## 🎉 Release v${{ steps.check_version.outputs.CMAKE_VERSION }}", etc.).

Locations:

- `.github/workflows/auto-release.yml:68`
- `.github/workflows/auto-release.yml:73`
- `.github/workflows/auto-release.yml:76`
- `.github/workflows/auto-release.yml:84`
- `.github/workflows/auto-release.yml:92`
- `.github/workflows/auto-release.yml:93`

### script-injection (severity: high)

Two run: blocks in functional-tests-tmpl.yml directly interpolate ${{ fromJSON(matrix.versions).cmake }} and ${{ fromJSON(matrix.versions).ninja }} into shell commands. The matrix.versions value is workflow-controlled and flows through YAML template substitution before the shell sees it. Affected steps: 'CMake version check' (CMAKE_REQUESTED_VER="${{ fromJSON(matrix.versions).cmake }}") and 'ninja version check' (NINJA_REQUESTED_VER="${{ fromJSON(matrix.versions).ninja }}").

Locations:

- `.github/workflows/functional-tests-tmpl.yml:42`
- `.github/workflows/functional-tests-tmpl.yml:65`

### unpinned-uses (severity: high)

All uses: references across workflow files use mutable version tags instead of pinned 40-character SHA commit hashes, making the workflows vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references: actions/checkout@v5, actions/download-artifact@v8, actions/setup-node@v6, actions/upload-artifact@v7, peter-evans/create-pull-request@v8.

Locations:

- `.github/workflows/auto-release.yml:27`
- `.github/workflows/build-test-tmpl.yml:25`
- `.github/workflows/build-test-tmpl.yml:28`
- `.github/workflows/build-test-tmpl.yml:31`
- `.github/workflows/build-test-tmpl.yml:55`
- `.github/workflows/build-test-tmpl.yml:65`
- `.github/workflows/functional-tests-tmpl.yml:30`
- `.github/workflows/functional-tests-tmpl.yml:33`
- `.github/workflows/sync-latest-and-tag.yml:27`

### missing-permissions (severity: medium)

build-test.yml has no top-level permissions: block and none of its jobs define job-level permissions. It triggers on push, pull_request, schedule, and workflow_dispatch, so it runs with the default (broad) repository permissions. The workflow should declare minimal permissions (e.g., contents: read) at the top level or per job.

Locations:

- `.github/workflows/build-test.yml:1`

### missing-permissions (severity: medium)

build-test-tmpl.yml (a workflow_call reusable workflow) has no top-level permissions: block and its only job (build_and_test) has no job-level permissions. Without explicit permissions, the job inherits whatever the caller grants, which may be overly broad. Minimal permissions (e.g., contents: read) should be declared.

Locations:

- `.github/workflows/build-test-tmpl.yml:1`

### missing-permissions (severity: medium)

functional-tests-tmpl.yml (a workflow_call reusable workflow) has no top-level permissions: block and its only job (test_user_provided_versions) has no job-level permissions. Without explicit permissions, the job inherits whatever the caller grants, which may be overly broad. Minimal permissions (e.g., contents: read) should be declared.

Locations:

- `.github/workflows/functional-tests-tmpl.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 7 findings across 5 workflow files:

1. script-injection (sync-latest-and-tag.yml): Moved ${{ inputs.tag_name }} and ${{ github.repository }} into env: blocks (TAG_NAME, GH_REPO) in 'Validate tag name', 'Create and push tag', and 'Summary' steps.

2. script-injection (auto-release.yml): Moved ${{ steps.check_version.outputs.CMAKE_VERSION }}, ${{ steps.check_version.outputs.SHOULD_RELEASE }}, and ${{ github.repository }} into env: blocks (CMAKE_VERSION, SHOULD_RELEASE, GH_REPO) in 'Create and push version tag' and 'Create release summary' steps.

3. script-injection (functional-tests-tmpl.yml): Moved ${{ fromJSON(matrix.versions).cmake }} and ${{ fromJSON(matrix.versions).ninja }} into env: blocks (CMAKE_REQUESTED_VER, NINJA_REQUESTED_VER) in 'CMake version check' and 'ninja version check' steps.

4. unpinned-uses: Pinned all 5 action references to full 40-char commit SHAs with tag comments: actions/checkout@fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09 (v5), actions/download-artifact@3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c (v8), actions/setup-node@249970729cb0ef3589644e2896645e5dc5ba9c38 (v6), actions/upload-artifact@043fb46d1a93c77aae656e7c1c64a875d1fc6a0a (v7), peter-evans/create-pull-request@5f6978faf089d4d20b00c7766989d076bb2fc7f1 (v8).

5. missing-permissions: Added 'permissions: contents: read' to build-test.yml, build-test-tmpl.yml, and functional-tests-tmpl.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in case statements in .github/workflows/functional-tests-tmpl.yml: (1) line 52: `case ${CMAKE_REQUESTED_VER} in` → `case "${CMAKE_REQUESTED_VER}" in`; (2) line 83: `case ${NINJA_REQUESTED_VER} in` → `case "${NINJA_REQUESTED_VER}" in`. These variables are sourced from matrix.versions (workflow-controllable data), so quoting them prevents shell metacharacter injection.

