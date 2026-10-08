Security Automation

 Overview

This tool is a Python-based security scanning utility for checking a code repository for common security issues.

The scanner covers four main areas:

1. SAST
2. Dependencies
3. Secrets
4. Infrastructure as Code

The scanner can use external security tools when they are installed and also provides built-in checks.

 Supported Security Tools

The scanner can integrate with:

- Bandit
- Semgrep
- Trivy
- pip-audit
- npm audit
- Gitleaks
- Checkov

The scanner does not require every external tool to be installed.

 Basic Usage

From this directory:

python3 security-scan.py --path ./app

For JSON output:

python3 security-scan.py --path ./app --format json

Save the results to a file:

python3 security-scan.py --path ./app --format json --output scan-results.json

Run only selected categories:

python3 security-scan.py --path ./app --only sast,secrets

Skip a category:

python3 security-scan.py --path ./app --skip iac

Use built-in checks only:

python3 security-scan.py --path ./app --builtin-only

 Severity

The scanner supports severity thresholds.

The default failure threshold is:

high

A scan can be configured with:

python3 security-scan.py --path ./app --fail-on critical

Exit Codes

0 = Scan passed
1 = Scan failed
2 = Invalid command or usage

The scanner is designed to fail closed when a scanner encounters an error unless --allow-errors is used.

 Security Design

The scanner follows a defence-in-depth approach.

External scanners provide deeper security checks where available, while built-in checks provide a baseline when external tools are unavailable.

The final result is combined into a single PASS or FAIL decision.

 Example

python3 security-scan.py --path ./application --format json --fail-on high

A successful scan returns exit code 0.

A scan containing findings at or above the configured severity returns exit code 1.

 Requirements

Python 3.8 or later is recommended.

External security tools are optional depending on the scan category.
