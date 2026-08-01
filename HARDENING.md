<!-- markdownlint-disable -->

# Hardening Report: lukka--get-cmake/v4.4.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **lukka--get-cmake/v4.4.2** was hardened automatically. 3 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files is pinned to a mutable tag rather than a full 40-character commit SHA. This exposes the workflow to supply-chain attacks if any referenced action's tag is moved or compromised. Affected references include: `actions/checkout@v7`, `actions/download-artifact@v8`, `actions/setup-node@v6`, `actions/upload-artifact@v7`, `peter-evans/create-pull-request@v8` (build-test-tmpl.yml); `actions/checkout@v7`, `actions/setup-node@v6` (functional-tests-tmpl.yml); `actions/checkout@v7` (auto-release.yml and sync-latest-and-tag.yml).

Locations:

- `.github/workflows/auto-release.yml:27`
- `.github/workflows/build-test-tmpl.yml:21`
- `.github/workflows/build-test-tmpl.yml:25`
- `.github/workflows/build-test-tmpl.yml:28`
- `.github/workflows/build-test-tmpl.yml:55`
- `.github/workflows/build-test-tmpl.yml:72`
- `.github/workflows/functional-tests-tmpl.yml:26`
- `.github/workflows/functional-tests-tmpl.yml:29`
- `.github/workflows/sync-latest-and-tag.yml:30`

### script-injection (severity: high)

Rule (a): Multiple `run:` blocks directly interpolate `${{ }}` expressions into shell scripts, allowing an attacker to inject arbitrary shell commands. In `auto-release.yml`, the 'Create and push version tag' step uses `TAG_NAME="v${{ steps.check_version.outputs.CMAKE_VERSION }}"` and `/repos/${{ github.repository }}/git/tags` directly inside the shell script. In `sync-latest-and-tag.yml`, the 'Validate tag name' and 'Create and push tag' steps use `TAG_NAME="${{ inputs.tag_name }}"` and `/repos/${{ github.repository }}/git/tags` directly in the shell. In `functional-tests-tmpl.yml`, the 'CMake version check' step uses `CMAKE_REQUESTED_VER="${{ fromJSON(matrix.versions).cmake }}"` and the 'ninja version check' step uses `NINJA_REQUESTED_VER="${{ fromJSON(matrix.versions).ninja }}"` directly in shell scripts — matrix values are workflow-controllable and flow through YAML template substitution before the shell sees them.

Locations:

- `.github/workflows/auto-release.yml:57`
- `.github/workflows/auto-release.yml:63`
- `.github/workflows/auto-release.yml:75`
- `.github/workflows/sync-latest-and-tag.yml:41`
- `.github/workflows/sync-latest-and-tag.yml:57`
- `.github/workflows/sync-latest-and-tag.yml:63`
- `.github/workflows/functional-tests-tmpl.yml:46`
- `.github/workflows/functional-tests-tmpl.yml:75`

### missing-permissions (severity: medium)

Three workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any of their jobs. Without explicit permissions, GitHub Actions defaults to the repository's default token permissions (which may be broad). `build-test.yml` triggers on `push`, `pull_request`, `schedule`, and `workflow_dispatch` with no permissions declared. `build-test-tmpl.yml` and `functional-tests-tmpl.yml` are reusable workflows (`workflow_call`) with no permissions declared on their jobs.

Locations:

- `.github/workflows/build-test.yml:1`
- `.github/workflows/build-test-tmpl.yml:1`
- `.github/workflows/functional-tests-tmpl.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions

**Notes:**

Fixed all three findings across 5 workflow files:

1. **unpinned-uses**: Pinned all action references to full 40-char SHAs with tag comments preserved:
   - actions/checkout@v7 → @3d3c42e5aac5ba805825da76410c181273ba90b1 (auto-release.yml, build-test-tmpl.yml, functional-tests-tmpl.yml, sync-latest-and-tag.yml)
   - actions/download-artifact@v8 → @3e5f45b2cfb9172054b4087a40e8e0b5a5461e7c (build-test-tmpl.yml)
   - actions/setup-node@v6 → @249970729cb0ef3589644e2896645e5dc5ba9c38 (build-test-tmpl.yml, functional-tests-tmpl.yml)
   - actions/upload-artifact@v7 → @043fb46d1a93c77aae656e7c1c64a875d1fc6a0a (build-test-tmpl.yml)
   - peter-evans/create-pull-request@v8 → @5f6978faf089d4d20b00c7766989d076bb2fc7f1 (build-test-tmpl.yml)

2. **script-injection**: Moved all ${{ }} expressions from run: blocks into env: blocks:
   - auto-release.yml: CMAKE_VERSION and GH_REPOSITORY env vars in 'Create and push version tag' and 'Create release summary' steps
   - sync-latest-and-tag.yml: TAG_NAME env var in 'Validate tag name', 'Create and push tag', and 'Summary' steps; GH_REPOSITORY in 'Create and push tag'
   - functional-tests-tmpl.yml: CMAKE_REQUESTED_VER env var in 'CMake version check'; NINJA_REQUESTED_VER in 'ninja version check'

3. **missing-permissions**: Added permissions blocks:
   - build-test.yml: top-level `permissions: {}` (caller workflow, delegates to reusable workflows)
   - build-test-tmpl.yml: job-level `permissions: contents: write, pull-requests: write` (needs write for commits and PR creation)
   - functional-tests-tmpl.yml: job-level `permissions: contents: read` (read-only access needed)

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two unquoted shell variable expansions in .github/workflows/functional-tests-tmpl.yml: (1) In the 'CMake version check' step, changed `case ${CMAKE_REQUESTED_VER} in` to `case "${CMAKE_REQUESTED_VER}" in` at line 44. (2) In the 'ninja version check' step, changed `case ${NINJA_REQUESTED_VER} in` to `case "${NINJA_REQUESTED_VER}" in` at line 72. Also repaired a file corruption that occurred during editing where the NINJA_VER assignment and case statement were accidentally merged onto one line.

