<!-- markdownlint-disable -->

# Hardening Report: lukka--get-cmake/v4.3.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lukka--get-cmake/v4.3.2** was hardened automatically. 10 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: ${{ ... }} expressions are interpolated directly inside run: shell command strings. In 'Create and push version tag' step: `TAG_NAME="v${{ steps.check_version.outputs.CMAKE_VERSION }}"`, `/repos/${{ github.repository }}/git/tags`, `-f message="Release $TAG_NAME - CMake ${{ steps.check_version.outputs.CMAKE_VERSION }}"`, `/repos/${{ github.repository }}/git/refs`. In 'Create release summary' step: `if [[ "${{ steps.check_version.outputs.SHOULD_RELEASE }}" == "true" ]]` and multiple `${{ steps.check_version.outputs.CMAKE_VERSION }}` interpolations. These allow YAML-template-substituted values to be parsed by the shell before quoting.

Locations:

- `.github/workflows/auto-release.yml:64`
- `.github/workflows/auto-release.yml:70`
- `.github/workflows/auto-release.yml:72`
- `.github/workflows/auto-release.yml:81`
- `.github/workflows/auto-release.yml:91`

### script-injection (severity: high)

Rule (a) violation: ${{ inputs.tag_name }} and ${{ github.repository }} are interpolated directly inside run: shell command strings. In 'Validate tag name' step: `TAG_NAME="${{ inputs.tag_name }}"`. In 'Create and push tag' step: `TAG_NAME="${{ inputs.tag_name }}"`, `/repos/${{ github.repository }}/git/tags`, `/repos/${{ github.repository }}/git/refs`. In 'Summary' step: `${{ inputs.tag_name }}` appears twice. The `inputs.tag_name` value is attacker-controlled via workflow_dispatch and is injected directly into shell commands without sanitization.

Locations:

- `.github/workflows/sync-latest-and-tag.yml:36`
- `.github/workflows/sync-latest-and-tag.yml:65`
- `.github/workflows/sync-latest-and-tag.yml:71`
- `.github/workflows/sync-latest-and-tag.yml:81`
- `.github/workflows/sync-latest-and-tag.yml:96`
- `.github/workflows/sync-latest-and-tag.yml:99`

### script-injection (severity: high)

Rule (a) violation: ${{ fromJSON(matrix.versions).cmake }} and ${{ fromJSON(matrix.versions).ninja }} are interpolated directly inside run: shell command strings. In 'CMake version check' step: `CMAKE_REQUESTED_VER="${{ fromJSON(matrix.versions).cmake }}"`. In 'ninja version check' step: `NINJA_REQUESTED_VER="${{ fromJSON(matrix.versions).ninja }}"`. The matrix.versions context is workflow-controllable and flows through YAML template substitution before the shell parses it.

Locations:

- `.github/workflows/functional-tests-tmpl.yml:42`
- `.github/workflows/functional-tests-tmpl.yml:74`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags instead of immutable 40-character commit SHAs, making the workflow vulnerable to supply-chain attacks if the tag is moved. Failing references: `actions/checkout@v5` (line 28).

Locations:

- `.github/workflows/auto-release.yml:28`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags instead of immutable 40-character commit SHAs. Failing references: `actions/checkout@v5` (line 25).

Locations:

- `.github/workflows/sync-latest-and-tag.yml:25`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags instead of immutable 40-character commit SHAs. Failing references: `actions/checkout@v5` (line 25), `actions/download-artifact@v8` (line 28), `actions/setup-node@v6` (line 30), `actions/upload-artifact@v7` (line 62), `peter-evans/create-pull-request@v7` (line 70).

Locations:

- `.github/workflows/build-test-tmpl.yml:25`
- `.github/workflows/build-test-tmpl.yml:28`
- `.github/workflows/build-test-tmpl.yml:30`
- `.github/workflows/build-test-tmpl.yml:62`
- `.github/workflows/build-test-tmpl.yml:70`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags instead of immutable 40-character commit SHAs. Failing references: `actions/checkout@v5` (line 30), `actions/setup-node@v6` (line 32).

Locations:

- `.github/workflows/functional-tests-tmpl.yml:30`
- `.github/workflows/functional-tests-tmpl.yml:32`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions (which may be read-write), violating the principle of least privilege.

Locations:

- `.github/workflows/build-test.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions.

Locations:

- `.github/workflows/build-test-tmpl.yml:1`

### missing-permissions (severity: medium)

The workflow file has no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, the workflow inherits the repository's default token permissions.

Locations:

- `.github/workflows/functional-tests-tmpl.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 10 findings across 5 workflow files:

1. auto-release.yml: Pinned actions/checkout@v5 to SHA fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09. Moved ${{ steps.check_version.outputs.CMAKE_VERSION }}, ${{ steps.check_version.outputs.SHOULD_RELEASE }}, and ${{ github.repository }} from run: shell strings into env: blocks.

2. sync-latest-and-tag.yml: Pinned actions/checkout@v5 to SHA fbc6f3992d24b796d5a048ff273f7fcc4a7b6c09. Moved ${{ inputs.tag_name }} and ${{ github.repository }} from run: shell strings into env: blocks.

3. functional-tests-tmpl.yml: Pinned actions/checkout@v5 and actions/setup-node@v6 to full SHAs. Moved ${{ fromJSON(matrix.versions).cmake }} and ${{ fromJSON(matrix.versions).ninja }} from run: shell strings into env: blocks. Added permissions: {} top-level block.

4. build-test-tmpl.yml: Pinned actions/checkout@v5, actions/download-artifact@v8, actions/setup-node@v6, actions/upload-artifact@v7, and peter-evans/create-pull-request@v7 to full commit SHAs. Added permissions: {} top-level block.

5. build-test.yml: Added permissions: {} top-level block.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in `.github/workflows/functional-tests-tmpl.yml`:
1. Line 52: Changed `case ${CMAKE_REQUESTED_VER} in` to `case "${CMAKE_REQUESTED_VER}" in` to properly quote the untrusted matrix-sourced variable.
2. Line 80: Changed `case ${NINJA_REQUESTED_VER} in` to `case "${NINJA_REQUESTED_VER}" in` to properly quote the untrusted matrix-sourced variable.
Both variables are sourced from `matrix.*` values (workflow-controllable) and were unquoted in `case` statements, which could allow shell metacharacters to alter script behavior.

### Iteration 3

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in bash regex patterns in .github/workflows/functional-tests-tmpl.yml. In both the 'CMake version check' and 'ninja version check' steps, replaced `[[ "$VAR" =~ .*${EXPECTED_VER}.* ]]` with `[[ "$VAR" == *"${EXPECTED_VER}"* ]]`. The glob pattern matching with double-quoted variable expansion treats the value as a literal string (no regex metacharacter interpretation), preventing injection of regex metacharacters through workflow-controlled matrix inputs. The fix preserves the same substring-containment check semantics.

