<!-- markdownlint-disable -->

# Hardening Report: tj-actions--pg-restore/v6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--pg-restore/v6** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The run: block directly interpolates GitHub Actions expressions into shell commands (sub-rule a). Specifically, `${{ inputs.options }}` is interpolated completely unquoted (also a sub-rule b violation), and `${{ inputs.database_url }}` and `${{ inputs.backup_file }}` are interpolated inside double-quoted strings. Any caller of this composite action can supply values containing shell metacharacters (`;`, `|`, `$(...)`, etc.) to achieve arbitrary command execution. The offending line is: `psql ${{ inputs.options }} -d "${{ inputs.database_url }}" < "${{ inputs.backup_file }}"`

Locations:

- `action.yml:24`

### unpinned-uses (severity: high)

The action uses `tj-actions/install-postgresql@v3`, which is pinned to a mutable version tag rather than an immutable 40-character commit SHA. This exposes the action to supply-chain attacks if the tag is moved to point to malicious code. It should be pinned to a full SHA, e.g. `tj-actions/install-postgresql@<40-char-sha> # v3`.

Locations:

- `action.yml:22`

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

**Fixes applied:** unpinned-uses, script-injection, static-inline-injection

**Notes:**

1. Pinned tj-actions/install-postgresql@v3 to full commit SHA a889ed6c6fa05022333ed4101295bb1d604f97a8 (kept # v3 comment for readability). 2. Moved all three ${{ inputs.* }} expressions out of the run: block into an env: block (INPUT_DATABASE_URL, INPUT_BACKUP_FILE, INPUT_OPTIONS). 3. For inputs.options (a list of extra psql flags), used the xargs-based tokenization pattern with a bash array to safely split the value into separate arguments while preserving quoting. 4. inputs.database_url and inputs.backup_file are single values referenced as double-quoted "$INPUT_DATABASE_URL" and "$INPUT_BACKUP_FILE" in the shell script.

