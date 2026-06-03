# Output Standardization and Centralization Guide

## Overview

Vulnerability scanners produce results in vendor-specific formats. Standardizing these outputs enables centralized reporting, trend analysis, compliance evidence, and integration with DevSecOps pipelines.

**Primary Standard:** SARIF (Static Analysis Results Interchange Format)
**Secondary Standards:** CycloneDX VEX, SCAP, STIX/TAXII

---

## SARIF — Static Analysis Results Interchange Format

**Standard Body:** OASIS Open
**Version:** 2.1.0 (current)
**Official Spec:** https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html
**Schema:** https://json.schemastore.org/sarif-2.1.0.json
**Validator:** https://sarifweb.azurewebsites.net
**GitHub Reference:** https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning

### What SARIF Is

SARIF is a JSON-based format for representing static analysis, DAST, and vulnerability scan results. It was originally designed for SAST tools but has been adopted broadly for security findings.

### SARIF Structure

```json
{
  "$schema": "https://json.schemastore.org/sarif-2.1.0.json",
  "version": "2.1.0",
  "runs": [
    {
      "tool": {
        "driver": {
          "name": "Nessus",
          "version": "10.x",
          "rules": [
            {
              "id": "plugin-97737",
              "name": "MS17-010 EternalBlue SMB RCE",
              "shortDescription": { "text": "CVE-2017-0144" },
              "helpUri": "https://www.tenable.com/plugins/nessus/97737",
              "properties": {
                "tags": ["security", "CVE-2017-0144", "CWE-119"]
              }
            }
          ]
        }
      },
      "results": [
        {
          "ruleId": "plugin-97737",
          "level": "error",
          "message": { "text": "The remote host is vulnerable to MS17-010 (EternalBlue)" },
          "locations": [
            {
              "physicalLocation": {
                "artifactLocation": { "uri": "192.168.1.10" }
              }
            }
          ],
          "properties": {
            "cvssScore": "9.8",
            "cvssVector": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:U/C:H/I:H/A:H"
          }
        }
      ]
    }
  ]
}
```

### SARIF Severity Levels

| SARIF Level | Meaning | Typical CVSS Range |
|---|---|---|
| `error` | Critical/High severity | 7.0–10.0 |
| `warning` | Medium severity | 4.0–6.9 |
| `note` | Low/Informational | 0.1–3.9 |
| `none` | Informational only | 0.0 |

### SARIF Converters by Tool

| Source Tool | Converter | Reference |
|---|---|---|
| Nessus (.nessus XML) | nessus-sarif-converter | https://github.com/Stift007/nessus-sarif-converter |
| OpenVAS (XML) | openvas-sarif | https://github.com/ShantoNoor/OpenVAS-to-SARIF |
| Qualys (XML) | qualys-sarif | Community converters |
| Bandit (Python SAST) | Built-in | `bandit -r . -f sarif -o results.sarif` |
| Semgrep | Built-in | `semgrep --sarif --output=results.sarif` |
| Trivy (container) | Built-in | `trivy image --format sarif myimage:latest` |
| Checkov (IaC) | Built-in | `checkov -d . --output sarif` |
| OWASP ZAP | Built-in | ZAP → Report → SARIF |
| Grype | Built-in | `grype myimage --output sarif` |

### SARIF Integration Platforms

| Platform | SARIF Support | Reference |
|---|---|---|
| GitHub Advanced Security | Native | https://docs.github.com/en/code-security/code-scanning |
| Azure DevOps | Native (SARIF upload) | https://learn.microsoft.com/en-us/azure/devops/pipelines/artifacts/sarif |
| GitLab | Native | https://docs.gitlab.com/ee/user/application_security/ |
| VS Code | SARIF Viewer extension | https://marketplace.visualstudio.com/items?itemName=MS-SarifVSCode.sarif-viewer |
| DefectDojo | Import SARIF | https://defectdojo.com |

---

## CycloneDX — SBOM + VEX

**Standard Body:** OWASP
**Official Reference:** https://cyclonedx.org
**Specification:** https://cyclonedx.org/specification/overview/

### What CycloneDX Is

CycloneDX provides:
- **SBOM (Software Bill of Materials):** Inventory of all components in software
- **VEX (Vulnerability Exploitability eXchange):** Vendor statements on whether a CVE affects their product

### VEX Use Case

When a CVE is published for a library (e.g., Log4j), VEX allows vendors to state:
- `not_affected`: The vulnerable component is present but not exploitable in this product
- `affected`: The product is affected
- `fixed`: The vulnerability has been remediated
- `under_investigation`: Still analyzing

**Example VEX Document:**
```json
{
  "bomFormat": "CycloneDX",
  "specVersion": "1.5",
  "vulnerabilities": [
    {
      "id": "CVE-2021-44228",
      "source": { "name": "NVD", "url": "https://nvd.nist.gov/vuln/detail/CVE-2021-44228" },
      "analysis": {
        "state": "not_affected",
        "justification": "code_not_reachable",
        "detail": "Log4j present but JNDI lookup disabled via configuration"
      }
    }
  ]
}
```

### CycloneDX Tools

| Tool | Use | Reference |
|---|---|---|
| Syft | SBOM generation | https://github.com/anchore/syft |
| cdxgen | SBOM for multiple ecosystems | https://github.com/CycloneDX/cdxgen |
| OWASP Dependency-Track | SBOM/VEX management platform | https://dependencytrack.org |
| CycloneDX CLI | Convert/validate BOM files | https://github.com/CycloneDX/cyclonedx-cli |

---

## SCAP — Security Content Automation Protocol

**Maintained by:** NIST
**Official Reference:** https://scap.nist.gov
**Current Version:** SCAP 1.3 / SCAP 2.0 (in development)

### SCAP Component Standards

| Component | Purpose | Reference |
|---|---|---|
| **XCCDF** | Checklist format for security configuration | https://csrc.nist.gov/projects/security-content-automation-protocol/scap-specifications |
| **OVAL** | Machine-readable vulnerability/config definitions | https://oval.mitre.org |
| **CVE** | Vulnerability naming | https://cve.mitre.org |
| **CPE** | Platform naming (product/version) | https://nvd.nist.gov/products/cpe |
| **CVSS** | Vulnerability scoring | https://www.first.org/cvss |
| **CCE** | Configuration item enumeration | https://csrc.nist.gov/projects/national-checklist-program |

### SCAP Validated Tools

Tools validated against SCAP can interoperate with NIST checklists:
- OpenSCAP (open source): https://www.open-scap.org
- Tenable Nessus (SCAP-validated)
- Qualys (SCAP-validated)
- **NVD SCAP Content:** https://nvd.nist.gov/ncp/repository

---

## STIX / TAXII — Threat Intelligence Sharing

**Standard Body:** OASIS Open
**Official Reference:** https://oasis-open.github.io/cti-documentation/
**STIX Version:** 2.1

### What STIX/TAXII Is

- **STIX (Structured Threat Information eXpression):** Format for representing threat intelligence (vulnerabilities, TTPs, threat actors, IoCs)
- **TAXII (Trusted Automated eXchange of Intelligence Information):** Protocol for sharing STIX data between organizations

### STIX Vulnerability Object

```json
{
  "type": "vulnerability",
  "id": "vulnerability--CVE-2021-44228",
  "name": "CVE-2021-44228",
  "external_references": [
    {
      "source_name": "cve",
      "external_id": "CVE-2021-44228"
    }
  ],
  "x_cvss": {
    "score": 10.0,
    "vector_string": "CVSS:3.1/AV:N/AC:L/PR:N/UI:N/S:C/C:H/I:H/A:H"
  }
}
```

### TAXII Feeds for Vulnerability Data

- CISA AIS (Automated Indicator Sharing): https://www.cisa.gov/ais
- AlienVault OTX: https://otx.alienvault.com
- MISP: https://www.misp-project.org

---

## Centralization Platforms

### DefectDojo (Open Source — Recommended)

**Reference:** https://github.com/DefectDojo/django-DefectDojo
**Docs:** https://defectdojo.com/documentation/

DefectDojo is the leading open-source vulnerability management platform. It aggregates results from multiple scanners into a unified view.

**Supported Importers:**
- Nessus (.nessus XML)
- OpenVAS (XML)
- Nexpose (XML)
- Qualys (XML)
- Burp Suite (XML)
- OWASP ZAP (XML, SARIF)
- Bandit (JSON, SARIF)
- Trivy (JSON, SARIF)
- Semgrep (SARIF)
- Checkmarx, Veracode, SonarQube, and 150+ more

**Key Features:**
- Deduplication across scanners
- Risk scoring and prioritization
- Jira/ServiceNow ticketing integration
- SARIF and CycloneDX export
- REST API for CI/CD integration
- Metrics dashboards

**Quick Start (Docker):**
```bash
git clone https://github.com/DefectDojo/django-DefectDojo
cd django-DefectDojo
./dc-up.sh
# Access at http://localhost:8080
```

---

### OWASP Dependency-Track (SCA/SBOM Focus)

**Reference:** https://dependencytrack.org
**GitHub:** https://github.com/DependencyTrack/dependency-track

Best for software supply chain / SCA results. Ingests CycloneDX SBOMs and automatically identifies components with known CVEs via NVD, OSV, and GitHub Advisory feeds.

---

### Centralization Architecture

```
                    ┌─────────────────────────────────────┐
                    │        SCANNER LAYER                │
                    │                                     │
  ┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐ │
  │ Tenable  │  │ OpenVAS  │  │ Nexpose  │  │ Qualys   │ │
  │ (.nessus)│  │  (XML)   │  │  (XML)   │  │  (XML)   │ │
  └────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘ │
       └─────────────┴──────────────┴──────────────┘       │
                           │                               │
                    ┌──────▼──────────────────────────┐    │
                    │   FORMAT CONVERSION LAYER        │    │
                    │  (SARIF / CycloneDX converters)  │    │
                    └──────┬──────────────────────────┘    │
                           │                               │
              ┌────────────▼───────────────┐               │
              │  CENTRALIZATION PLATFORM   │               │
              │  (DefectDojo / Dep-Track)  │               │
              └────────────┬───────────────┘               │
                           │                               │
         ┌─────────────────┼──────────────────────┐       │
         ▼                 ▼                        ▼       │
  ┌─────────────┐  ┌──────────────┐  ┌────────────────┐   │
  │    SIEM     │  │  Ticketing   │  │   Dashboards   │   │
  │  (Sentinel  │  │(Jira/Service │  │(Power BI/Grafana│   │
  │   Splunk)   │  │    Now)      │  │    /native)     │   │
  └─────────────┘  └──────────────┘  └────────────────┘   │
                                                           │
                    └─────────────────────────────────────┘
```

---

## Recommended Integration Pipelines

### CI/CD Pipeline (DevSecOps)

```yaml
# Example GitHub Actions with SARIF upload
name: Security Scan
on: [push, pull_request]
jobs:
  scan:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # SAST - Semgrep
      - name: Semgrep SAST
        run: |
          pip install semgrep
          semgrep --sarif --output=semgrep.sarif .

      # SCA - Trivy
      - name: Trivy Dependency Scan
        run: |
          trivy fs --format sarif --output trivy.sarif .

      # Upload to GitHub Advanced Security
      - name: Upload SARIF results
        uses: github/codeql-action/upload-sarif@v2
        with:
          sarif_file: '*.sarif'
```

### SIEM Integration (Microsoft Sentinel Example)

- Tenable.io → Azure Sentinel connector: https://learn.microsoft.com/en-us/azure/sentinel/data-connectors/tenable-io
- Qualys → Sentinel: https://learn.microsoft.com/en-us/azure/sentinel/data-connectors/qualys-vm-knowledgebase
- Custom: Use Logic Apps or Azure Functions to poll scanner APIs and push to Sentinel

---

## Official References

| Resource | URL |
|---|---|
| SARIF Specification v2.1.0 | https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html |
| SARIF Validator | https://sarifweb.azurewebsites.net |
| GitHub SARIF Support | https://docs.github.com/en/code-security/code-scanning/integrating-with-code-scanning/sarif-support-for-code-scanning |
| CycloneDX Specification | https://cyclonedx.org/specification/overview/ |
| CycloneDX VEX | https://cyclonedx.org/capabilities/vex/ |
| OWASP Dependency-Track | https://dependencytrack.org |
| DefectDojo | https://defectdojo.com |
| SCAP | https://scap.nist.gov |
| OVAL | https://oval.mitre.org |
| OpenSCAP | https://www.open-scap.org |
| STIX/TAXII | https://oasis-open.github.io/cti-documentation/ |
| MISP | https://www.misp-project.org |
| NVD SCAP Content | https://nvd.nist.gov/ncp/repository |
