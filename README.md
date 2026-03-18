# TraceCert

**Making AI-generated software verifiable.**

TraceCert turns the AI coding process itself into a continuous audit. Instead of auditing *after* you build, the audit *is* the build.

It generates cryptographically signed certification levels (1–4) with full provenance, so regulated industries (healthcare, finance, government, legal) can trust AI-built software.

## Features

- `tracecert scan .` — One-command scan + certification
- Built on real **Agent Trace** format (Cursor 2026 standard)
- Graduated certification levels:
  - **Level 1**: AI-Built + basic quality
  - **Level 2**: Privacy & Security Verified
  - **Level 3**: Domain Grade (HIPAA plugin included)
  - **Level 4**: Enterprise (future)
- Real-time HIPAA analysis with risk level and actionable notes
- Supports `--json`, `--max-files`, `--exclude`, `--no-hipaa`
- Installable CLI tool

## Installation

```bash
pip install -e .

## Usage

# Basic scan and certify
tracecert scan .

# With options
tracecert scan . --max-files 500 --exclude tests,docs,venv
tracecert scan . --json
tracecert scan . --no-hipaa

# Certify an existing trace file
tracecert certify path/to/trace.jsonl

## Example Output (Healthcare)
Level Achieved      : 3
AI-Generated        : 99.0% (631 lines)
Compliance Score    : 60/100
HIPAA Risk Level    : HIGH
HIPAA Score         : 40/100
Notes               : ⚠️ 14 high-risk phrases detected | Significant improvements needed before healthcare use

## License

**Proprietary License** — All Rights Reserved.

This project is **not open source**. You are welcome to use it for personal, internal evaluation, or non-commercial purposes.

Commercial use, redistribution, resale, or offering this tool (or derivatives) as a product/service to third parties requires explicit written permission from TraceCert.

See the [LICENSE](LICENSE) file for full details.

## Commercial / Enterprise Plans

We are actively developing **TraceCert Enterprise**, which will include:

- Level 4 Enterprise Certification (SOC 2 pathway, external auditor support)
- Advanced domain plugins (Finance, Legal, Government, FDA)
- Team collaboration & audit history
- Automated compliance report generation (PDF + evidence packs)
- Cloud-hosted verification service
- Priority support and custom compliance mappings

If you're interested in early access, enterprise licensing, or would like to discuss using TraceCert in a commercial setting, please reach out @ar8itrt0r@gmail.com.

---

**Built with the mission of making AI-generated software trusted in regulated industries.**

Inspired by ideas shared on X.