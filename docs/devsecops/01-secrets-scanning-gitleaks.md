# Stage 1 — Secrets Scanning (Gitleaks)

Reference for the first gate of the DevSecOps pipeline. Everything you need to
re-run, re-baseline, or understand this stage lives here.

## What this gate does

Detects hard-coded secrets (API keys, tokens, private keys, passwords) in the
code **and the entire git history**, on every push and pull request. If a secret
is found, the build fails and blocks the merge (`--exit-code 1`).

## Where Gitleaks runs (layered, like real teams)

| Context | Where | When |
|---------|-------|------|
| **Pre-commit** | Your machine, before the commit exists | Every `git commit` (staged changes) — the earliest, cheapest catch |
| **CI · PR diff** | Fresh Ubuntu runner | Every pull request — scans only the PR's new commits (fast; fails only on secrets the PR adds) |
| **CI · full history** | Fresh Ubuntu runner | Push to default branch + weekly schedule — full-history safety net |
| **Local (manual)** | Your own machine | Ad-hoc checks, or (re)generating the baseline |
| **Docker** | A container | Alternative to installing the binary |

Gitleaks only *reads* the repo — it doesn't live inside it. The workflow file is
stored in the repo but executes on the runner.

Why not scan full history on every PR? Because it is slow and would re-flag old,
already-triaged findings. PRs scan just `base..head`; the full sweep runs on a
schedule instead.

## Running Gitleaks locally

### Install

```bash
# Linux / macOS (Homebrew)
brew install gitleaks

# Linux (binary) — replace VERSION with the latest from the releases page
VERSION=8.30.1
curl -sSfL "https://github.com/gitleaks/gitleaks/releases/download/v${VERSION}/gitleaks_${VERSION}_linux_x64.tar.gz" -o gitleaks.tar.gz
tar -xzf gitleaks.tar.gz gitleaks && sudo install -m 0755 gitleaks /usr/local/bin/gitleaks

# Docker (no install)
docker run -v "$(pwd):/repo" zricethezav/gitleaks:latest git /repo
```

```powershell
# Windows (PowerShell) — download the release zip
$VERSION = "8.30.1"
curl.exe -sSL -o gitleaks.zip "https://github.com/gitleaks/gitleaks/releases/download/v$VERSION/gitleaks_${VERSION}_windows_x64.zip"
Expand-Archive .\gitleaks.zip -DestinationPath .\gitleaks-bin -Force
# then call .\gitleaks-bin\gitleaks.exe
```

Tip: on Windows you can also use `winget install gitleaks.gitleaks` or
`scoop install gitleaks` if you have those package managers.

### Scan

```bash
# scan the full git history (same as CI)
gitleaks git . --config .gitleaks.toml --verbose

# scan only the working files (fast, no history)
gitleaks dir . --config .gitleaks.toml --verbose
```

## The baseline (`.gitleaksignore`)

A brand-new scan on a legacy repo returns many findings that are intentional
(test fixtures, seeded demo data, deliberate training vulns). We accept the
current state as a **baseline** so the gate only fails on *new* secrets.

We use `.gitleaksignore` (a list of **fingerprints**, not secret values) rather
than `--baseline-path` (a full report that would contain the real secrets in
plaintext). Gitleaks reads `.gitleaksignore` from the repo root automatically —
no workflow flag needed.

A fingerprint looks like: `commit:file:rule-id:line`.

### (Re)generating the baseline

Do this only when you deliberately want to re-accept the current findings.

```bash
# Linux / macOS
gitleaks git . --report-format json --report-path report.json
jq -r '.[].Fingerprint' report.json > .gitleaksignore
rm report.json          # contains real secrets — never commit it
```

```powershell
# Windows (PowerShell)
.\gitleaks-bin\gitleaks.exe git . --report-format json --report-path report.json
(Get-Content .\report.json -Raw | ConvertFrom-Json).Fingerprint | Set-Content .gitleaksignore
Remove-Item report.json                          # contains real secrets — never commit it
```

Then commit **only** `.gitleaksignore`.

## Pre-commit hook (shift left to the laptop)

The cheapest place to catch a secret is before the commit exists. We use the
[pre-commit](https://pre-commit.com/) framework with `.pre-commit-config.yaml`.

One-time setup per clone:

```bash
pip install pre-commit     # or: pipx install pre-commit
pre-commit install         # installs the hook into .git/hooks
```

After that, every `git commit` runs gitleaks on the staged changes and blocks the
commit if it finds a secret. It reuses this repo's `.gitleaks.toml` and
`.gitleaksignore`, so local and CI stay in sync. Emergency bypass (use sparingly):

```bash
SKIP=gitleaks git commit -m "..."
```

## Triaging a new finding

When the gate fails on a *new* finding, decide:

1. **Real secret** → remove it from code, rotate/revoke it, and (if it entered
   history) purge it with `git filter-repo` or BFG. Never just delete the line.
2. **Intentional / false positive** → add the fingerprint to `.gitleaksignore`,
   or add a `# gitleaks:allow` comment on that line, or a justified `[[allowlists]]`
   entry in `.gitleaks.toml`.

Never allowlist something you haven't personally verified is safe.

## Files that make up this stage

- `.github/workflows/secrets-scan.yml` — the CI jobs (PR-diff + full-history).
- `.pre-commit-config.yaml` — local pre-commit hook.
- `.gitleaks.toml` — config (extends the default ruleset + hybrid path allowlist).
- `.gitleaksignore` — the baseline of accepted historical findings.
