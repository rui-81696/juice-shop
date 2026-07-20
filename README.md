# Security Lab — OWASP Juice Shop (DevSecOps + Pentesting)

A hands-on learning project built on top of OWASP Juice Shop, split into two
complementary tracks: **defending** the application with a secure CI/CD pipeline,
and **attacking** it as a pentester and documenting each finding.

> Based on [OWASP Juice Shop](https://github.com/juice-shop/juice-shop) (MIT License).

## Project tracks

| Track | Goal | Lives in |
|-------|------|----------|
| 🛡️ **DevSecOps** | Build a secure CI/CD pipeline with security gates across the SDLC | this repo root + `.github/workflows/` |
| 🗡️ **Pentesting** | Exploit a running instance and write up each finding (attack → defense) | [`pentest/`](./pentest/) |

The two tracks close a loop: every vulnerability found in the pentesting track should
map back to a pipeline gate that would detect or mitigate it.

---

## 🛡️ DevSecOps pipeline

Secure CI/CD pipeline integrating security gates across the SDLC: secrets scanning,
SAST, SCA, container scanning and DAST.

### Pipeline stages
_(em construção — diagrama na Fase 7)_

### Tools
| Stage | Tool | Status |
|-------|------|--------|
| Secrets scanning | Gitleaks | ✅ |
| SAST | Semgrep | 🔜 |
| SCA | Trivy | 🔜 |
| Container scan | Trivy + Hadolint | 🔜 |
| DAST | OWASP ZAP | 🔜 |
| IaC security (extra) | Checkov / tfsec + Terraform | 🔜 |

> **Extra phase — Infrastructure as Code.** Beyond the 5 core stages: provision
> deploy infrastructure with Terraform and gate it with an IaC security scanner
> (Checkov / tfsec), so misconfigurations are caught before the infra is created.

---

## 🗡️ Pentesting

Offensive-security track: attacking a local instance of Juice Shop and documenting
each finding as a professional-style writeup (impact, PoC, remediation, and the
DevSecOps gate that would have caught it).

All testing targets **my own local instance only** — see [`pentest/README.md`](./pentest/README.md)
for methodology, target setup, and the writeups index.
