# TraceCert

**Making AI-generated software verifiable.**

TraceCert turns the AI coding process into a continuous, auditable record. Instead of auditing *after* you build, the audit *is* the build. It scans your codebase, parses Agent Trace logs, and produces a cryptographically signed certification manifest with graduated verification levels — built for regulated industries where trust in AI-generated code must be demonstrable.

---

## Table of Contents

- [How It Works](#how-it-works)
- [Certification Levels](#certification-levels)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [CLI Reference](#cli-reference)
- [Agent Trace Format](#agent-trace-format)
- [Policy-as-Code](#policy-as-code)
- [CI/CD Integration](#cicd-integration)
- [Example Output](#example-output)
- [Plugins](#plugins)
- [Enterprise](#enterprise)
- [License](#license)

---

## How It Works

```
Your codebase / Agent Trace log
         |
         v
  tracecert scan .
         |
         v
  Parse trace records --> Run verification plugins --> Sign manifest
         |
         v
  .tracecert/cert-manifest.json   (signed, tamper-evident)
```

TraceCert reads Agent Trace JSONL files — logs produced by AI coding tools like Cursor — and runs them through a pipeline of verification plugins. The result is a signed manifest you can store in your repo, ship with your artifact, or gate on in CI.

---

## Certification Levels

| Level | Name | What it verifies |
|-------|------|-----------------|
| **1** | AI-Built | Trace parsed, AI provenance recorded, basic quality checks |
| **2** | Security Verified | Secret detection, OWASP no-secrets check, provenance chain |
| **3** | Domain Grade | HIPAA risk scoring, SOC 2 TSC alignment, AI quality (slop) detection |
| **4** | Enterprise | Combined SOC 2 + HIPAA signal; requires Level 3 thresholds met |

Levels are additive — achieving Level 3 means Levels 1 and 2 were also verified.

---

## Installation

**Basic install:**

```bash
pip install -e .
```

**With PDF export and YAML policy support (recommended):**

```bash
pip install -e ".[enterprise]"
```

**Requirements:** Python 3.9+

---

## Quick Start

**1. Scan your project directory:**

```bash
tracecert scan .
```

**2. Certify an existing Agent Trace file:**

```bash
tracecert certify examples/sample-agent-trace.jsonl
```

**3. Export a PDF report and SVG badge:**

```bash
tracecert scan . --export-pdf
```

**4. Run in CI with a policy gate:**

```bash
tracecert scan . --policy .tracecert/policy.yaml
```

The signed manifest is written to `.tracecert/cert-manifest.json` by default.

---

## CLI Reference

### `tracecert scan [PATH]`

Scans a directory, auto-generates a trace, and certifies it.

```
tracecert scan [PATH] [OPTIONS]

Arguments:
  PATH                  Directory to scan (default: current directory)

Options:
  -o, --output PATH     Output path for manifest (default: .tracecert/cert-manifest.json)
  --level INT           Verification level requested, 1-4 (default: 3)
  --no-hipaa            Disable the HIPAA/Healthcare plugin
  --sast                Run SAST scanners - Semgrep/Bandit/Bearer (enterprise, slow)
  --export-pdf          Export PDF report and SVG badge (requires fpdf2)
  --format TEXT         Output format: text | json | pdf (default: text)
  --policy PATH         Path to a policy.yaml file
  --remediate           Print auto-remediation suggestions
  --model TEXT          AI model name for provenance (e.g. claude-3.5-sonnet)
  --prompt-file PATH    Prompt file to hash for provenance
  --json                Output the full JSON manifest to stdout
  --max-files INT       Maximum files to scan (default: 5000)
  --exclude TEXT        Comma-separated directories to exclude
```

**Examples:**

```bash
# Scan current directory, all defaults
tracecert scan .

# Exclude test and build directories, cap at 500 files
tracecert scan . --max-files 500 --exclude tests,docs,venv,build

# Output full JSON manifest
tracecert scan . --json

# Disable HIPAA plugin (non-healthcare projects)
tracecert scan . --no-hipaa

# Attach model provenance
tracecert scan . --model claude-3.5-sonnet --prompt-file prompts/system.txt

# Full enterprise run with PDF and policy enforcement
tracecert scan . --export-pdf --policy .tracecert/policy.yaml --remediate
```

---

### `tracecert certify TRACE_FILE`

Certifies an existing Agent Trace JSONL file without re-scanning.

```
tracecert certify TRACE_FILE [OPTIONS]

Arguments:
  TRACE_FILE            Path to an Agent Trace .jsonl file

Options:
  (same as scan, minus PATH-specific options)
```

**Examples:**

```bash
# Certify a trace file at default level
tracecert certify examples/sample-agent-trace.jsonl

# Certify and emit JSON
tracecert certify my-trace.jsonl --json

# Certify with healthcare plugin disabled
tracecert certify my-trace.jsonl --no-hipaa
```

---

### `tracecert pr-check`

Posts a compliance report to a GitHub PR and sets a commit status. Intended for CI.

```
tracecert pr-check [OPTIONS]

Options:
  --manifest PATH       Manifest to evaluate (default: .tracecert/cert-manifest.json)
  --pr TEXT             PR number to comment on
  --sha TEXT            Commit SHA for the status check
  --repo TEXT           owner/repo (default: GITHUB_REPOSITORY env var)
  --threshold INT       Minimum score to pass (default: 90)
  --policy PATH         Path to a policy.yaml file
```

**Example:**

```bash
tracecert pr-check --threshold 85 --pr 42 --sha abc1234
```

Exits with code `1` if the score is below threshold.

---

## Agent Trace Format

TraceCert reads the **Agent Trace** JSONL format. Each line is a JSON record describing one AI coding action.

**Minimal record:**

```jsonl
{"version": "0.1.0", "id": "rec_001", "timestamp": "2026-03-17T12:00:00Z", "files": [{"path": "app/main.py", "ai_generated": true}]}
```

**Full record with provenance:**

```jsonl
{
  "version": "0.1.0",
  "id": "rec_001",
  "timestamp": "2026-03-17T12:00:00Z",
  "vcs": {"type": "git", "revision": "a1b2c3d4"},
  "tool": {"name": "claude"},
  "files": [
    {"path": "app/main.py", "ai_generated": true, "lines": 120},
    {"path": "app/utils.py", "ai_generated": false, "lines": 45}
  ],
  "metadata": {
    "model": "claude-3.5-sonnet",
    "prompt_hash": "sha256:abcd...",
    "content_hash": "sha256:efgh..."
  }
}
```

**Fields:**

| Field | Required | Description |
|-------|----------|-------------|
| `version` | Yes | Trace format version |
| `id` | Yes | Unique record ID |
| `timestamp` | Yes | ISO 8601 UTC timestamp |
| `files` | Yes | Array of files touched |
| `files[].path` | Yes | Relative file path |
| `files[].ai_generated` | Yes | Whether this file was AI-generated |
| `files[].lines` | No | Estimated line count (default: 50) |
| `metadata.model` | No | AI model used |
| `metadata.prompt_hash` | No | SHA-256 of the prompt for provenance |
| `vcs.revision` | No | Git commit SHA |

See [examples/sample-agent-trace.jsonl](examples/sample-agent-trace.jsonl) for a working example.

---

## Policy-as-Code

Create `.tracecert/policy.yaml` in your repo to enforce gates automatically during `tracecert scan` and `tracecert pr-check`.

**Example policy:**

```yaml
# Minimum verification score to pass (0-100)
min_score: 85

# Block CI if any secrets are found in trace content
block_on_secrets: true

# Custom rules — evaluated against flattened attestation fields
# Supported ops: gte (>=), lte (<=), eq (==)
rules:
  # Require at least 70/100 on AI quality (slop) detection
  - name: min_ai_quality
    field: slop_slop_score
    op: gte
    value: 70

  # Require SOC 2 TSC alignment score >= 60
  - name: soc2_tsc_alignment
    field: soc2_soc2_score
    op: gte
    value: 60

  # Require PHI risk score >= 65 for healthcare projects
  - name: phi_risk_threshold
    field: healthcare_compliance_score
    op: gte
    value: 65

  # Zero high-severity SAST findings (uncomment if using --sast)
  # - name: no_high_sast
  #   field: sast_high_severity
  #   op: lte
  #   value: 0
```

See [.tracecert/policy.example.yaml](.tracecert/policy.example.yaml) for the full annotated template.

When a policy violation is detected, the CLI prints the violations and exits with code `1`, blocking the CI merge.

---

## CI/CD Integration

### GitHub Actions

Add this step after your build to certify every push and PR:

```yaml
- name: Install TraceCert
  run: pip install -e ".[enterprise]"

- name: Certify AI trace
  run: tracecert certify examples/sample-agent-trace.jsonl --policy .tracecert/policy.yaml

- name: Upload manifest
  uses: actions/upload-artifact@v4
  with:
    name: tracecert-manifest
    path: .tracecert/cert-manifest.json
```

**Gate a PR with a compliance score threshold:**

```yaml
- name: PR compliance check
  env:
    GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
  run: |
    tracecert pr-check \
      --pr ${{ github.event.pull_request.number }} \
      --sha ${{ github.sha }} \
      --threshold 85
```

See [.github/workflows/](.github/workflows/) for the full workflow definitions included in this repo.

---

## Example Output

```
================================================================================
TraceCert Level 3 Verification -- Manifest Signed
================================================================================
Verification Level  : 3
Levels Verified     : [1, 2, 3]
AI-Generated        : 99.0% (631 lines)
Verification Score  : 88/100
Records Processed   : 5
Unique Files        : 5
Secrets Found       : 0
HIPAA-ready checks  : True
PHI Risk Score      : 72/100  [PHI risk: MEDIUM per our analysis]
HIPAA-ready         : True  (matches our HIPAA-ready rule set)
SOC2 TSC Alignment  : 78/100  [B]  (aligns with SOC 2 TSC mapping)
AI Quality Grade    : B  (slop score 74)
Provenance Entries  : 5
Signature Type      : MVP
Output File         : .tracecert/cert-manifest.json
================================================================================
```

**JSON output** (`--json`):

```json
{
  "version": "1.0.0",
  "spec": "tracecert+v1",
  "generated_at": "2026-03-17T12:00:05Z",
  "certification": {
    "highest_level": 3,
    "levels_verified": [1, 2, 3],
    "summary": {
      "ai_percentage": 99.0,
      "compliance_score": 88,
      "secrets_found": 0,
      "hipaa_ready": true,
      "soc2_ready": true
    }
  },
  "attestations": [...],
  "provenance": [...],
  "signature": {"type": "mvp", "digest": "sha256:..."}
}
```

---

## Plugins

Plugins implement domain-specific Level 3 verification. They run automatically unless disabled.

| Plugin | Domain key | Flag to disable | What it checks |
|--------|------------|-----------------|----------------|
| **Healthcare** | `healthcare` | `--no-hipaa` | PHI risk phrases, HIPAA §164.312 mapping, compliance score |
| **SOC 2** | `soc2` | always on | SOC 2 Trust Services Criteria alignment (CC6, CC7, CC8, A1, C1, PI1, P1) |
| **Slop** | `slop` | always on | AI output quality grade (A-F), detects over-generated boilerplate |
| **SAST** | `sast` | opt-in (`--sast`) | Runs Semgrep/Bandit/Bearer; reports high/medium/low findings |

Custom plugins can be built by subclassing `tracecert.plugins.base.Plugin`.

---

## Enterprise

TraceCert Enterprise extends the open platform with:

- **Level 4 Certification** — Combined SOC 2 TSC + HIPAA §164.312 signal in a single attestation
- **Advanced domain plugins** — Finance, Legal, Government, FDA
- **SAST integration** — Semgrep, Bandit, Bearer via `--sast`
- **PDF reports + SVG badges** — via `--export-pdf` (requires `pip install -e ".[enterprise]"`)
- **PR bot** — Automated GitHub PR comments and commit status checks
- **Team collaboration and audit history**
- **Cloud-hosted verification service**
- **Custom compliance mappings and priority support**

For early access or enterprise licensing, contact **ar8itrt0r@gmail.com**.

---

## License

**Proprietary — All Rights Reserved.**

This project is not open source. Personal and internal evaluation use is permitted. Commercial use, redistribution, or offering this tool as a product or service to third parties requires explicit written permission.

See the [LICENSE](LICENSE) file for full terms.

---

*Built with the mission of making AI-generated software trusted in regulated industries.*
