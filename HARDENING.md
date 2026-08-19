<!-- markdownlint-disable -->

# Hardening Report: lukka--get-cmake/v4.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lukka--get-cmake/v4.3.0** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use tag-based (non-SHA-pinned) `uses:` references, making them vulnerable to supply-chain attacks if the referenced action tags are moved or compromised. Failing references:
- `actions/checkout@v5` (tag, not a SHA)
- `actions/download-artifact@v8` (tag, not a SHA)
- `actions/setup-node@v6` (tag, not a SHA)
- `actions/upload-artifact@v7` (tag, not a SHA)
- `peter-evans/create-pull-request@v7` (tag, not a SHA)
All should be pinned to full 40-character commit SHAs.

Locations:

- `.github/workflows/auto-release.yml:27`
- `.github/workflows/build-test-tmpl.yml:25`
- `.github/workflows/build-test-tmpl.yml:28`
- `.github/workflows/build-test-tmpl.yml:31`
- `.github/workflows/build-test-tmpl.yml:62`
- `.github/workflows/build-test-tmpl.yml:87`
- `.github/workflows/functional-tests-tmpl.yml:31`
- `.github/workflows/functional-tests-tmpl.yml:33`
- `.github/workflows/sync-latest-and-tag.yml:26`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell commands, enabling script injection. Any expression interpolated before the shell parses the command can inject arbitrary shell metacharacters.

**auto-release.yml** — 'Create and push version tag' step (sub-rule a):
- `TAG_NAME="v${{ steps.check_version.outputs.CMAKE_VERSION }}"`
- `/repos/${{ github.repository }}/git/tags`
- `-f message="Release $TAG_NAME - CMake ${{ steps.check_version.outputs.CMAKE_VERSION }}"`
- `/repos/${{ github.repository }}/git/refs`

**auto-release.yml** — 'Create release summary' step (sub-rule a):
- `if [[ "${{ steps.check_version.outputs.SHOULD_RELEASE }}" == "true" ]];`
- `echo "## 🎉 Release v${{ steps.check_version.outputs.CMAKE_VERSION }} created!"`

**sync-latest-and-tag.yml** — 'Validate tag name' step (sub-rule a):
- `TAG_NAME="${{ inputs.tag_name }}"` — `inputs.tag_name` is attacker-controlled via workflow_dispatch

**sync-latest-and-tag.yml** — 'Create and push tag' step (sub-rule a):
- `TAG_NAME="${{ inputs.tag_name }}"`
- `/repos/${{ github.repository }}/git/tags`
- `/repos/${{ github.repository }}/git/refs`

**sync-latest-and-tag.yml** — 'Summary' step (sub-rule a):
- `echo "- ✓ Tag \`${{ inputs.tag_name }}\` created"`
- `echo "- \`@${{ inputs.tag_name }}\` - for this specific version"`

**functional-tests-tmpl.yml** — 'CMake version check' step (sub-rule a):
- `CMAKE_REQUESTED_VER="${{ fromJSON(matrix.versions).cmake }}"`

**functional-tests-tmpl.yml** — 'ninja version check' step (sub-rule a):
- `NINJA_REQUESTED_VER="${{ fromJSON(matrix.versions).ninja }}"`

Locations:

- `.github/workflows/auto-release.yml:63`
- `.github/workflows/auto-release.yml:68`
- `.github/workflows/auto-release.yml:70`
- `.github/workflows/auto-release.yml:80`
- `.github/workflows/auto-release.yml:89`
- `.github/workflows/auto-release.yml:90`
- `.github/workflows/sync-latest-and-tag.yml:37`
- `.github/workflows/sync-latest-and-tag.yml:64`
- `.github/workflows/sync-latest-and-tag.yml:69`
- `.github/workflows/sync-latest-and-tag.yml:79`
- `.github/workflows/sync-latest-and-tag.yml:93`
- `.github/workflows/sync-latest-and-tag.yml:96`
- `.github/workflows/functional-tests-tmpl.yml:44`
- `.github/workflows/functional-tests-tmpl.yml:72`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no job-level `permissions:` blocks on their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) permissions, which vary by repository settings.

- `build-test-tmpl.yml`: The `build_and_test` job has no `permissions:` block.
- `functional-tests-tmpl.yml`: The `test_user_provided_versions` job has no `permissions:` block.
- `build-test.yml`: No top-level `permissions:` and no job-level `permissions:` on any job.

Each file should declare a top-level `permissions: {}` (deny-all default) and grant only the specific scopes required.

Locations:

- `.github/workflows/build-test-tmpl.yml:1`
- `.github/workflows/functional-tests-tmpl.yml:1`
- `.github/workflows/build-test.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 5 workflow files:

**unpinned-uses**: Pinned all 5 action references to full 40-char SHAs with tag comments:
- actions/checkout@v5 → @93cb6efe18208431cddfb8368fd83d5badbf9bfd # v5
- actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c # v8
- actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 # v6
- actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a # v7
- peter-evans/create-pull-request@v7 → @22a9089034f40e5a961c8808d113e2c98fb63676 # v7

**script-injection**: Moved all ${{ }} expressions from run: shell scripts into step env: blocks, referencing them as plain shell variables ($VAR_NAME) in the scripts. Affected steps: 'Create and push version tag' and 'Create release summary' in auto-release.yml; 'Validate tag name', 'Create and push tag', and 'Summary' in sync-latest-and-tag.yml; 'CMake version check' and 'ninja version check' in functional-tests-tmpl.yml.

**missing-permissions**: Added `permissions: contents: read` at job level to build-test-tmpl.yml and functional-tests-tmpl.yml; added top-level `permissions: {}` to build-test.yml.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted variable expansions in `case` statements in `.github/workflows/functional-tests-tmpl.yml`:
1. Line 52: Changed `case ${CMAKE_REQUESTED_VER} in` to `case "${CMAKE_REQUESTED_VER}" in`
2. Line 80: Changed `case ${NINJA_REQUESTED_VER} in` to `case "${NINJA_REQUESTED_VER}" in`

Both variables hold caller-controlled values from `${{ fromJSON(matrix.versions).cmake }}` and `${{ fromJSON(matrix.versions).ninja }}` respectively. Quoting them in the `case` statement prevents shell metacharacters from being interpreted as shell syntax.

