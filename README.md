# DevSecOps Pipeline — OWASP Juice Shop

Secure CI/CD pipeline built on top of OWASP Juice Shop, integrating
security gates across the SDLC: secrets scanning, SAST, SCA,
container scanning and DAST.

> Based on [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) (MIT License).

## Pipeline stages
_(em construção — diagrama na Fase 7)_

## Tools
| Stage | Tool | Status |
|-------|------|--------|
| Secrets scanning | Gitleaks | 🔜 |
| SAST | Semgrep | 🔜 |
| SCA | Trivy | 🔜 |
| Container scan | Trivy + Hadolint | 🔜 |
| DAST | OWASP ZAP | 🔜 |