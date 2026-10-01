<!-- markdownlint-disable -->

# Hardening Report: tj-actions--pg-restore/v6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **tj-actions--pg-restore/v6.0** was hardened automatically. 5 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The `run:` block in action.yml directly interpolates GitHub Actions expressions into shell commands (rule a). Three inputs are interpolated: `${{ inputs.options }}` is completely unquoted (allowing shell metacharacter injection), and `${{ inputs.database_url }}` and `${{ inputs.backup_file }}` are double-quoted but still directly substituted into the shell command string before the shell parses it. An attacker-controlled caller can supply values containing shell metacharacters (`;`, `|`, `$(...)`, backticks, etc.) to achieve arbitrary command execution. The offending line is: `psql ${{ inputs.options }} -d "${{ inputs.database_url }}" < "${{ inputs.backup_file }}"`

Fix: move all inputs into `env:` variables and reference them as quoted shell variables, e.g.:
```yaml
env:
  PSQL_OPTIONS: ${{ inputs.options }}
  DATABASE_URL: ${{ inputs.database_url }}
  BACKUP_FILE: ${{ inputs.backup_file }}
run: |
  psql ${PSQL_OPTIONS:+"$PSQL_OPTIONS"} -d "$DATABASE_URL" < "$BACKUP_FILE"
```

Locations:

- `action.yml:22`

### unpinned-uses (severity: high)

The composite action step uses `tj-actions/install-postgresql@v3`, which is pinned to a mutable version tag (`@v3`) rather than an immutable 40-character commit SHA. A tag can be moved to point to a different (potentially malicious) commit at any time, enabling a supply-chain attack. Fix: pin to a full SHA, e.g. `tj-actions/install-postgresql@<40-char-sha> # v3`.

Locations:

- `action.yml:19`

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

1. Pinned `tj-actions/install-postgresql@v3` to full SHA `a889ed6c6fa05022333ed4101295bb1d604f97a8 # v3`. 2. Moved all three inputs (`options`, `database_url`, `backup_file`) from inline `${{ }}` expressions in the `run:` block into an `env:` map. The `options` input (a whitespace-separated list of psql flags) is safely tokenized using the xargs+while-read-NUL pattern into a bash array with a required non-empty guard, then expanded as `"${opts[@]}"`. The `database_url` and `backup_file` inputs are referenced as double-quoted shell variables `"$DATABASE_URL"` and `"$BACKUP_FILE"`.

