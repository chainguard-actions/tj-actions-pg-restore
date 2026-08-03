<!-- markdownlint-disable -->

# Hardening Report: tj-actions--pg-restore/v6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--pg-restore/v6.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates three untrusted `inputs.*` expressions into a shell command string (sub-rule a). `${{ inputs.options }}` is injected unquoted, and `${{ inputs.database_url }}` and `${{ inputs.backup_file }}` are injected inside double-quoted strings — all before the shell processes them. An attacker controlling these inputs can inject arbitrary shell commands. The offending line is: `psql ${{ inputs.options }} -d "${{ inputs.database_url }}" < "${{ inputs.backup_file }}"`

Locations:

- `action.yml:22`

### unpinned-uses (severity: high)

Multiple `uses:` references are pinned to mutable tags or version strings instead of immutable 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved.

action.yml:
- `tj-actions/install-postgresql@v3`

.github/workflows/codacy-analysis.yml:
- `actions/checkout@v4`
- `codacy/codacy-analysis-cli-action@v4.3.0`
- `github/codeql-action/upload-sarif@v3`

.github/workflows/rebase.yml:
- `actions/checkout@v4`
- `cirrus-actions/rebase@1.8`

.github/workflows/sync-release-version.yml:
- `actions/checkout@v4`
- `tj-actions/release-tagger@v4`
- `tj-actions/sync-release-version@v13`
- `tj-actions/git-cliff@v1`
- `peter-evans/create-pull-request@v5.0.2`

.github/workflows/test.yml:
- `actions/checkout@v4`

.github/workflows/update-readme.yml:
- `actions/checkout@v4.1.1`
- `tj-actions/auto-doc@v3.4.1`
- `tj-actions/remark@v3`
- `tj-actions/verify-changed-files@v17`
- `peter-evans/create-pull-request@v5.0.2`

Locations:

- `action.yml:20`
- `.github/workflows/codacy-analysis.yml:28`
- `.github/workflows/codacy-analysis.yml:33`
- `.github/workflows/codacy-analysis.yml:47`
- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:14`
- `.github/workflows/sync-release-version.yml:10`
- `.github/workflows/sync-release-version.yml:12`
- `.github/workflows/sync-release-version.yml:13`
- `.github/workflows/sync-release-version.yml:18`
- `.github/workflows/sync-release-version.yml:22`
- `.github/workflows/test.yml:30`
- `.github/workflows/update-readme.yml:10`
- `.github/workflows/update-readme.yml:13`
- `.github/workflows/update-readme.yml:15`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:29`

### missing-permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and no individual job within any of these files defines a `permissions:` block. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (e.g., write access to contents and pull requests). Affected files: codacy-analysis.yml, rebase.yml, sync-release-version.yml, test.yml, update-readme.yml.

Locations:

- `.github/workflows/codacy-analysis.yml:1`
- `.github/workflows/rebase.yml:1`
- `.github/workflows/sync-release-version.yml:1`
- `.github/workflows/test.yml:1`
- `.github/workflows/update-readme.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.options }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.database_url }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:28`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.backup_file }}" appears directly in run: block of step ""; move to env: map

Locations:

- `action.yml:28`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all findings across action.yml and 5 workflow files:

1. script-injection (action.yml): Moved inputs.options, inputs.database_url, and inputs.backup_file out of the run: block into the step's env: map. inputs.options is split with 'read -ra opts' since it's a list of psql options; database_url and backup_file are single values referenced as double-quoted env vars.

2. unpinned-uses: Pinned all 13 action references across action.yml and all 5 workflow files to their full 40-character commit SHAs, preserving the original tag in a trailing comment.

3. missing-permissions: Added top-level permissions blocks to all 5 workflow files with minimal required permissions (codacy-analysis.yml: contents:read + security-events:write; rebase.yml: contents:write + pull-requests:read; sync-release-version.yml: contents:write + pull-requests:write; test.yml: contents:read; update-readme.yml: contents:write + pull-requests:write).

