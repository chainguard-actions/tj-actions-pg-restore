<!-- markdownlint-disable -->

# Hardening Report: tj-actions--pg-restore/v6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--pg-restore/v6** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): action.yml line 27 directly interpolates three untrusted inputs inside a run: shell command — `${{ inputs.options }}` (also unquoted, violating sub-rule (b)), `${{ inputs.database_url }}`, and `${{ inputs.backup_file }}`. An attacker who controls these inputs (e.g. via workflow_dispatch or a calling workflow) can inject arbitrary shell commands. The offending line is: `psql ${{ inputs.options }} -d "${{ inputs.database_url }}" < "${{ inputs.backup_file }}"`

Locations:

- `action.yml:27`

### unpinned-uses (severity: high)

Multiple uses: references are pinned to mutable tags or version strings instead of full 40-character commit SHAs, making the action vulnerable to supply-chain attacks if those tags are moved or hijacked.

action.yml:
  - tj-actions/install-postgresql@v3 (line 22)

.github/workflows/codacy-analysis.yml:
  - actions/checkout@v4 (line 30)
  - codacy/codacy-analysis-cli-action@v4.3.0 (line 34)
  - github/codeql-action/upload-sarif@v3 (line 48)

.github/workflows/rebase.yml:
  - actions/checkout@v4 (line 10)
  - cirrus-actions/rebase@1.8 (line 14)

.github/workflows/sync-release-version.yml:
  - actions/checkout@v4 (line 10)
  - tj-actions/release-tagger@v4 (line 12)
  - tj-actions/sync-release-version@v13 (line 13)
  - tj-actions/git-cliff@v1 (line 18)
  - peter-evans/create-pull-request@v5.0.2 (line 20)

.github/workflows/test.yml:
  - actions/checkout@v4 (line 37, line 53)

.github/workflows/update-readme.yml:
  - actions/checkout@v4.1.1 (line 11)
  - tj-actions/auto-doc@v3.4.1 (line 15)
  - tj-actions/remark@v3 (line 18)
  - tj-actions/verify-changed-files@v17 (line 21)
  - peter-evans/create-pull-request@v5.0.2 (line 33)

Locations:

- `action.yml:22`
- `.github/workflows/codacy-analysis.yml:30`
- `.github/workflows/codacy-analysis.yml:34`
- `.github/workflows/codacy-analysis.yml:48`
- `.github/workflows/rebase.yml:10`
- `.github/workflows/rebase.yml:14`
- `.github/workflows/sync-release-version.yml:10`
- `.github/workflows/sync-release-version.yml:12`
- `.github/workflows/sync-release-version.yml:13`
- `.github/workflows/sync-release-version.yml:18`
- `.github/workflows/sync-release-version.yml:20`
- `.github/workflows/test.yml:37`
- `.github/workflows/test.yml:53`
- `.github/workflows/update-readme.yml:11`
- `.github/workflows/update-readme.yml:15`
- `.github/workflows/update-readme.yml:18`
- `.github/workflows/update-readme.yml:21`
- `.github/workflows/update-readme.yml:33`

### missing-permissions (severity: medium)

None of the five workflow files define a top-level permissions: key, and no job within them defines a job-level permissions: key either. Without explicit permissions, workflows inherit the default repository permissions (which may be write-all), granting jobs more access than they need. Each workflow should declare minimal required permissions (e.g. `permissions: contents: read`).

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

1. script-injection/static-inline-injection (action.yml): Moved inputs.options, inputs.database_url, and inputs.backup_file out of the run: shell string into the step's env: block. The options input (a list of psql flags) is tokenized safely with xargs into a bash array to prevent both injection and argument-boundary issues.

2. unpinned-uses: Pinned all 13 action references across action.yml and all 5 workflow files to their full 40-character commit SHAs, preserving the original tag in a trailing comment.

3. missing-permissions: Added top-level permissions: blocks with minimal required permissions to all 5 workflow files (codacy-analysis.yml: contents:read + security-events:write; rebase.yml: contents:write + pull-requests:read; sync-release-version.yml: contents:write + pull-requests:write; test.yml: contents:read; update-readme.yml: contents:write + pull-requests:write).

