<!-- markdownlint-disable -->

# Hardening Report: tj-actions--pg-restore/v6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--pg-restore/v6.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block in action.yml directly interpolates multiple ${{ inputs.* }} expressions into a shell command string (sub-rule a). Specifically: `psql ${{ inputs.options }} -d "${{ inputs.database_url }}" < "${{ inputs.backup_file }}"`. An attacker-controlled caller can supply values containing shell metacharacters (`;`, `|`, `$(...)`, etc.) that will be executed by the shell before any quoting takes effect. These inputs must be moved to env: variables and then double-quoted in the shell script.

Locations:

- `action.yml:22`

### unpinned-uses (severity: high)

Multiple `uses:` references across action.yml and workflow files use mutable tags or version strings instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the referenced tag is moved or overwritten.

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
- `actions/checkout@v4` (appears twice)

.github/workflows/update-readme.yml:
- `actions/checkout@v4.1.1`
- `tj-actions/auto-doc@v3.4.1`
- `tj-actions/remark@v3`
- `tj-actions/verify-changed-files@v17`
- `peter-evans/create-pull-request@v5.0.2`

Locations:

- `action.yml:18`
- `.github/workflows/codacy-analysis.yml:29`
- `.github/workflows/codacy-analysis.yml:34`
- `.github/workflows/codacy-analysis.yml:49`
- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:15`
- `.github/workflows/sync-release-version.yml:9`
- `.github/workflows/sync-release-version.yml:11`
- `.github/workflows/sync-release-version.yml:13`
- `.github/workflows/sync-release-version.yml:17`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/test.yml:30`
- `.github/workflows/test.yml:53`
- `.github/workflows/update-readme.yml:9`
- `.github/workflows/update-readme.yml:13`
- `.github/workflows/update-readme.yml:15`
- `.github/workflows/update-readme.yml:17`
- `.github/workflows/update-readme.yml:32`

### missing-permissions (severity: medium)

None of the 5 workflow files under .github/workflows/ declare a top-level `permissions:` key, and no individual job within any of these files declares a `permissions:` key either. Without explicit permissions, workflows run with the default (often broad) token permissions, violating the principle of least privilege. Each workflow should declare minimal required permissions (e.g., `permissions: read-all` or specific scopes like `contents: read`).

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

1. script-injection/static-inline-injection (action.yml line 28): Moved inputs.options, inputs.database_url, and inputs.backup_file from inline ${{ }} expressions in run: block to env: map variables. The options input (a list of psql flags) is tokenized via xargs into a bash array to preserve argument boundaries while preventing injection.

2. unpinned-uses: Pinned all 13 action references across action.yml and all 5 workflow files to full 40-character commit SHAs, preserving original tags as inline comments.

3. missing-permissions: Added top-level permissions blocks to all 5 workflow files with minimal required scopes (codacy-analysis: contents:read + security-events:write; rebase: contents:write + pull-requests:read; sync-release-version: contents:write + pull-requests:write; test: contents:read; update-readme: contents:write + pull-requests:write).

