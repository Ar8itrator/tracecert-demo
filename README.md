# TraceCert

**Making AI-generated software verifiable.**

TraceCert turns the AI coding process itself into a continuous audit. Instead of auditing *after* you build, the audit *is* the build.

It generates cryptographically signed certification levels with full provenance, so regulated industries (healthcare, finance, government) can trust AI-built software.

## Features

- One-command scan + certification: `tracecert scan .`
- Built on real **Agent Trace** format
- Graduated certification levels (1–4)
- Real-time **HIPAA Grade** analysis (Level 3)
- Supports `--json`, `--max-files`, `--exclude`, `--no-hipaa`

## Quick Start

```bash
# Clone the public demo repo
git clone https://github.com/Ar8itrator/tracecert.git
cd tracecert

# Install
pip install -e .

# Scan and certify your project
tracecert scan .

## Example Output
Level Achieved      : 3
AI-Generated        : 99.0% (631 lines)
Compliance Score    : 60/100
HIPAA Risk Level    : HIGH
HIPAA Score         : 40/100
Notes               : ⚠️ 14 high-risk phrases detected | Significant improvements needed before healthcare use