<!-- markdownlint-disable -->

# Hardening Report: tj-actions--pg-restore/v6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--pg-restore/v6** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates untrusted inputs into a shell command string via ${{ inputs.options }}, ${{ inputs.database_url }}, and ${{ inputs.backup_file }}. An attacker controlling these inputs (e.g. via a calling workflow) can inject arbitrary shell commands. Sub-rule (a): direct expression interpolation inside a run: block. Offending line: `psql ${{ inputs.options }} -d "${{ inputs.database_url }}" < "${{ inputs.backup_file }}"`

Locations:

- `action.yml:22`

### unpinned-uses (severity: high)

Multiple uses: references across action.yml and workflow files use mutable tags instead of full 40-character SHA commit hashes, making the action vulnerable to supply-chain attacks if the referenced tag is moved or overwritten. Failing references include: action.yml: tj-actions/install-postgresql@v3; codacy-analysis.yml: actions/checkout@v4, codacy/codacy-analysis-cli-action@v4.3.0, github/codeql-action/upload-sarif@v3; rebase.yml: actions/checkout@v4, cirrus-actions/rebase@1.8; sync-release-version.yml: actions/checkout@v4, tj-actions/release-tagger@v4, tj-actions/sync-release-version@v13, tj-actions/git-cliff@v1, peter-evans/create-pull-request@v5.0.2; test.yml: actions/checkout@v4 (×2); update-readme.yml: actions/checkout@v4.1.1, tj-actions/auto-doc@v3.4.1, tj-actions/remark@v3, tj-actions/verify-changed-files@v17, peter-evans/create-pull-request@v5.0.2.

Locations:

- `action.yml:19`
- `.github/workflows/codacy-analysis.yml:30`
- `.github/workflows/codacy-analysis.yml:35`
- `.github/workflows/codacy-analysis.yml:51`
- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:14`
- `.github/workflows/sync-release-version.yml:9`
- `.github/workflows/sync-release-version.yml:12`
- `.github/workflows/sync-release-version.yml:14`
- `.github/workflows/sync-release-version.yml:18`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/test.yml:27`
- `.github/workflows/test.yml:51`
- `.github/workflows/update-readme.yml:10`
- `.github/workflows/update-readme.yml:14`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:20`
- `.github/workflows/update-readme.yml:32`

### missing-permissions (severity: medium)

None of the five workflow files define a top-level permissions: key, and no individual job within any of these files defines a job-level permissions: key. Without explicit permissions, workflows run with the default (potentially write) token permissions, violating the principle of least privilege.

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

1. script-injection / static-inline-injection (action.yml line 28): Moved ${{ inputs.options }}, ${{ inputs.database_url }}, and ${{ inputs.backup_file }} into an env: block as INPUT_OPTIONS, INPUT_DATABASE_URL, INPUT_BACKUP_FILE. Since 'options' is a list of psql flags, used 'read -ra opts <<< "$INPUT_OPTIONS"' to safely split and expand it as an array.

2. unpinned-uses: Pinned all 13 action references across action.yml and all 5 workflow files to their full 40-character commit SHAs, preserving the original tag in a trailing comment.

3. missing-permissions: Added top-level permissions blocks to all 5 workflow files with minimal required permissions: codacy-analysis.yml (contents: read, security-events: write), rebase.yml (contents: write, pull-requests: read), sync-release-version.yml (contents: write, pull-requests: write), test.yml (contents: read), update-readme.yml (contents: write, pull-requests: write).

