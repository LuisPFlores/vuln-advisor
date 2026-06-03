# Arcus — Vulnerability Assessment & Threat Alert Analysis Agent

> An AI-powered expert agent for vulnerability assessment, classification, scoring, remediation guidance, Microsoft Defender POC planning, and Defender threat alert analysis. Built for security engineers, SOC analysts, developers, and IT administrators.

---

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Repository Structure](#repository-structure)
- [Functions](#functions)
  - [1. Vulnerability Classification](#1-vulnerability-classification)
  - [2. CVSS / CVE / CWE Analysis](#2-cvss--cve--cwe-analysis)
  - [3. Vulnerability Management Life Cycle](#3-vulnerability-management-life-cycle)
  - [4. Vendor Tool Analysis](#4-vendor-tool-analysis)
  - [5. Output Standardization](#5-output-standardization)
  - [6. AI-Powered Tools](#6-ai-powered-tools)
  - [7. Staying Current](#7-staying-current)
  - [8. Defender Alert Analysis](#8-defender-alert-analysis)
- [How to Use the Agent](#how-to-use-the-agent)
- [Official References](#official-references)

---

## Overview

**Arcus** is a structured AI cybersecurity agent with three modes:

- **Mode 1 — Vulnerability Assessment:** Every response follows a consistent 10-step analysis pattern:

```
Question received
  └── Classify vulnerability type
        └── Map CWE root cause
              └── Identify CVE(s) + CVSS score
                    └── Check CISA KEV status
                          └── Check MSRC advisory
                                └── Evaluate tool coverage (6 scanners in order)
                                      └── Identify applicable AI tools
                                            └── Map to lifecycle phase
                                                  └── Recommend output format
                                                        └── Cite official references
```

- **Mode 2 — Microsoft Defender POC Advisor:** Describe a security scenario → get a product recommendation + step-by-step POC deployment plan.
- **Mode 3 — Defender Alert Analysis:** Paste a Defender alert or alert ID → get MITRE ATT&CK mapping, Composite Severity Score, response actions, and a Sentinel KQL hunting query.

---

## Installation

### Prerequisites

- [Git](https://git-scm.com) — to clone the repository
- [GitHub Copilot](https://github.com/features/copilot) — with a Copilot Individual, Business, or Enterprise subscription
- [Visual Studio Code](https://code.visualstudio.com) with the [GitHub Copilot extension](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot) installed, **or** access to [github.com/copilot](https://github.com/copilot)
- No additional runtime dependencies; all knowledge base files are plain Markdown

### Clone the Repository

```bash
git clone https://github.com/LuisPFlores/arcus.git
cd arcus
```

### Option 1 — Repository-Level Instructions (Copilot in VS Code)

This approach automatically activates Arcus whenever you open this repository in VS Code with GitHub Copilot.

1. Copy the system prompt into the GitHub Copilot instructions file for the repo:

   ```bash
   # From the repo root
   mkdir -p .github
   cp agent/AGENT.md .github/copilot-instructions.md
   ```

2. Open the repository in VS Code:

   ```bash
   code .
   ```

3. Open **Copilot Chat** (`Ctrl+Alt+I` / `⌃⌘I`) and start a conversation. Copilot will automatically apply the Arcus instructions to every response in this workspace.

> **How it works:** VS Code reads `.github/copilot-instructions.md` and prepends its content to every Copilot Chat request made within the workspace. No manual prompting required.

**Reference:** [GitHub Docs — Adding repository custom instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)

---

### Option 2 — Personal Instructions (Copilot in VS Code, all workspaces)

This approach loads Arcus as your personal Copilot assistant across every project.

1. Open VS Code **Settings** (`Ctrl+,` / `⌘,`)
2. Search for `github.copilot.chat.codeGeneration.instructions`
3. Click **Edit in settings.json** and add:

   ```json
   "github.copilot.chat.codeGeneration.instructions": [
     {
       "file": "/absolute/path/to/arcus/agent/AGENT.md"
     }
   ]
   ```

   Replace `/absolute/path/to/arcus` with the actual path where you cloned the repo.

4. Save and reload VS Code. Arcus instructions are now active in all Copilot Chat sessions.

**Reference:** [GitHub Docs — Specifying instructions for Copilot to follow](https://docs.github.com/en/copilot/customizing-copilot/adding-personal-custom-instructions-for-github-copilot)

---

### Option 3 — GitHub Copilot on GitHub.com

Use Arcus directly in your browser through [github.com/copilot](https://github.com/copilot):

1. Open [github.com/copilot](https://github.com/copilot) and start a new chat.
2. In the chat input, attach the system prompt using the **Attach file** button or paste the contents of `agent/AGENT.md` as context.
3. Alternatively, reference this repository directly by typing:

   ```
   Use the instructions in https://github.com/LuisPFlores/arcus/blob/master/agent/AGENT.md
   ```

4. Ask your vulnerability assessment or Defender alert question.

> **Tip:** Pin this conversation or save the prompt for repeated use.

---

### Option 4 — GitHub Copilot Extensions (Enterprise / Advanced)

For organizations using [GitHub Copilot Extensions](https://docs.github.com/en/copilot/building-copilot-extensions/about-building-copilot-extensions):

1. Publish Arcus as a GitHub App with a Copilot agent by using `agent/AGENT.md` as the system prompt.
2. Install the extension from your GitHub organization's Copilot settings.
3. Invoke Arcus in Copilot Chat using `@arcus [your question]`.

**Reference:** [GitHub Docs — Building Copilot Extensions](https://docs.github.com/en/copilot/building-copilot-extensions/about-building-copilot-extensions)

---

### Knowledge Base Reference

All knowledge base files are self-contained Markdown — no build step, no package manager, no environment variables required:

| Folder | Purpose |
|---|---|
| `agent/` | System prompt (`AGENT.md`), agent config, skill definition |
| `vulnerability/` | CVE/CWE/CVSS reference, classification taxonomy, lifecycle guide |
| `tools/` | Scanner comparisons, AI tool catalog, output format standards |
| `defender/` | Defender POC playbooks, alert analysis guide |

---

## Repository Structure

```
arcus/
├── README.md                                    ← This file (root only)
├── agent/
│   ├── .agent.json                              ← OpenCode agent configuration
│   ├── AGENT.md                                 ← Full system prompt and behavior rules
│   └── SKILL.md                                 ← OpenCode skill definition (3 modes)
├── vulnerability/
│   ├── vulnerability-classifications.md         ← 8 vulnerability classification categories
│   ├── cvss-cve-cwe-reference.md                ← CVSS v2/v3.1/v4.0, CVE, CWE deep dive
│   └── vulnerability-management-lifecycle.md    ← 6-phase lifecycle guide with AI tools per phase
├── tools/
│   ├── vendor-tool-comparison.md                ← 6 traditional scanner tools compared
│   ├── ai-powered-tools.md                      ← 13 AI-powered tools + Defender aggregation
│   └── output-standardization.md               ← SARIF, CycloneDX, SCAP, DefectDojo
└── defender/
    ├── microsoft-defender-poc.md                ← 8 POC playbooks + scenario-to-product matrix
    └── defender-alert-analysis.md              ← Graph API, MITRE ATT&CK, CSS model, KQL queries
```

---

## Functions

### 1. Vulnerability Classification

**Reference file:** `vulnerability-classifications.md`

The agent classifies every finding into one or more of the following 8 categories:

| # | Category | Description | Key CWEs | OWASP Mapping |
|---|---|---|---|---|
| 1 | **Misconfigurations / Weak Configurations** | Improperly configured services, permissions, or security controls | CWE-16, CWE-732, CWE-1188 | A05:2021 |
| 2 | **Network Vulnerabilities** | Open ports, unencrypted protocols, weak firewall rules, ARP/DNS attacks | CWE-311, CWE-319, CWE-300 | A02:2021 |
| 3 | **Poor Patch Management** | Missing OS/app patches, EOL software, outdated libraries | CWE-1104, CWE-1352 | A06:2021 |
| 4 | **Application Flaws** | SQLi, XSS, CSRF, SSRF, insecure deserialization, broken auth | CWE-79, CWE-89, CWE-78, CWE-918 | A01–A10:2021 |
| 5 | **Design Flaws** | Architectural weaknesses, no encryption at rest, hardcoded secrets | CWE-259, CWE-311, CWE-250 | A04:2021 |
| 6 | **Default Installations / Configurations** | Default credentials, open admin interfaces, unchanged vendor defaults | CWE-1188, CWE-521, CWE-258 | A05:2021 |
| 7 | **Operating System Flaws** | Kernel vulnerabilities, LPE bugs, unpatched OS components | CWE-119, CWE-269, CWE-416 | N/A (OS-level) |
| 8 | **Zero-Day Vulnerabilities** | Publicly unknown or unpatched flaws actively exploited | Varies | Varies |

Each classification includes:
- Representative CVE examples with CVSS scores
- CIS Benchmark and NIST SP 800-30 alignment
- Scanner detection guidance per tool
- Applicable compliance framework mapping

**Official Reference:** NIST SP 800-30 Rev 1 — https://csrc.nist.gov/publications/detail/sp/800-30/rev-1/final

---

### 2. CVSS / CVE / CWE Analysis

**Reference file:** `cvss-cve-cwe-reference.md`

The agent provides full scoring and identification analysis for every vulnerability.

#### CVSS — Common Vulnerability Scoring System

Supported versions:

| Version | Status | Notes |
|---|---|---|
| CVSS v2.0 | Legacy | Deprecated, shown for historical CVEs |
| CVSS v3.1 | Current | Widely used across all major platforms |
| CVSS v4.0 | Latest (Nov 2023) | New metric groups (CVSS-B, BT, BE, BTE), OT/ICS support |

**Severity Ratings:**

| Score | Severity | Recommended Remediation SLA |
|---|---|---|
| 0.0 | None | Informational |
| 0.1–3.9 | Low | 180 days |
| 4.0–6.9 | Medium | 90 days |
| 7.0–8.9 | High | 30 days |
| 9.0–10.0 | Critical | 15 days (immediate if in KEV) |

**Official Reference:** https://www.first.org/cvss/

#### CVE — Common Vulnerabilities and Exposures

- Maintained by MITRE, sponsored by CISA/DHS
- Format: `CVE-YEAR-SEQUENCE` (e.g., `CVE-2021-44228`)
- Enriched with CVSS scores, CWE mappings, and CPE data at NVD

**Official References:**
- https://cve.mitre.org
- https://nvd.nist.gov

#### CWE — Common Weakness Enumeration

- Taxonomy of software and hardware weakness types
- Includes the **CWE Top 25 Most Dangerous Software Weaknesses** (updated annually)
- Format: `CWE-ID` (e.g., `CWE-89` — SQL Injection)

**Top 5 CWEs (2024):**

| Rank | CWE | Name |
|---|---|---|
| 1 | CWE-79 | Cross-Site Scripting (XSS) |
| 2 | CWE-787 | Out-of-bounds Write |
| 3 | CWE-89 | SQL Injection |
| 4 | CWE-416 | Use After Free |
| 5 | CWE-78 | OS Command Injection |

**Official Reference:** https://cwe.mitre.org/top25/

#### Relationship Model

```
CWE (weakness root cause)
  └── leads to →
        CVE (specific vulnerability instance)
          └── scored by →
                CVSS (severity + exploitability)
                  └── prioritized via →
                        EPSS (exploitation probability)
                        CISA KEV (confirmed active exploitation)
```

---

### 3. Vulnerability Management Life Cycle

**Reference file:** `vulnerability-management-lifecycle.md`

The agent maps every question to a phase in the 6-phase life cycle and provides deep step-by-step guidance, recommendations, tooling, and decision criteria for each.

```
┌──────────────────────────────────────────────────────────────────────────┐
│                                                                          │
│   ┌──────────┐   ┌────────────┐   ┌─────┐   ┌──────────┐   ┌─────────┐ │
│   │ DISCOVER │──▶│ PRIORITIZE │──▶│ ACT │──▶│ REASSESS │──▶│ IMPROVE │ │
│   └──────────┘   └────────────┘   └─────┘   └──────────┘   └────┬────┘ │
│        ▲                                                          │      │
│        │                   ┌──────────┐                          │      │
│        └───────────────────│  REPORT  │◀─────────────────────────┘      │
│                            └──────────┘                                  │
└──────────────────────────────────────────────────────────────────────────┘
```

#### Phase 1: DISCOVER — 5 Steps

| Step | Action | Key Recommendation |
|---|---|---|
| 1.1 | Define scope | Obtain written authorization; integrate CMDB as asset source of truth |
| 1.2 | Asset discovery and classification | Tag every asset with owner, criticality tier, data classification, and compliance scope |
| 1.3 | Select scanning method | Prefer credentialed scans; use passive-only for OT/ICS; store credentials in PAM |
| 1.4 | Execute scans | Whitelist scanner IP in IPS; log all scan activity; target >95% coverage of Tier 1–2 assets |
| 1.5 | Validate results | Eliminate false positives via credentialed scanning and vendor advisory cross-reference |

**Asset Criticality Tiers:**

| Tier | Criticality | Examples | Scan Frequency |
|---|---|---|---|
| 1 — Critical | Business-stopping | Payment systems, AD, prod DBs, internet-facing services | Continuous or weekly |
| 2 — High | Significant impact | Internal servers, ERP, HR, build systems | Bi-weekly |
| 3 — Medium | Moderate impact | Workstations, printers, VoIP | Monthly |
| 4 — Low | Minimal impact | Dev/test, IoT sensors | Quarterly |

#### Phase 2: PRIORITIZE — 5 Steps

| Step | Action | Key Recommendation |
|---|---|---|
| 2.1 | Apply multi-factor risk scoring | Composite score: CVSS × Asset Criticality × EPSS × Exposure × Compensating Controls |
| 2.2 | Assess each prioritization input | Evaluate CVSS, EPSS, KEV status, asset criticality, exposure, and compensating controls |
| 2.3 | Classify by priority tier (P0–P5) | Never use CVSS alone — a Critical on an isolated test server < High on internet-facing prod |
| 2.4 | Assign ownership | Every vulnerability must have a named, accountable owner; integrate with Jira/ServiceNow |
| 2.5 | Document risk acceptance | Time-bound expiry mandatory; max 90 days for Critical; requires named executive approver |

**Priority Tier Definitions:**

| Tier | Criteria | SLA |
|---|---|---|
| **P0 — Emergency** | Critical + High EPSS + In KEV, or active exploitation confirmed | 24–48 hours |
| **P1 — Urgent** | Critical + High EPSS, or High + In KEV | 7 calendar days |
| **P2 — High** | Critical without exploit, or High + High EPSS | 15 calendar days |
| **P3 — Medium** | High + Low EPSS, or Medium CVSS | 30–90 calendar days |
| **P4 — Low** | Low/Medium + Low EPSS + Not in KEV | 180 calendar days |
| **P5 — Info** | Informational, no direct remediation | Review at next cycle |

#### Phase 3: ACT — 4 Steps + 3 Special Scenarios

| Step | Action | Key Recommendation |
|---|---|---|
| 3.1 | Select remediation strategy | Patching preferred; for Microsoft CVEs always check MSRC for official mitigations/workarounds before applying generic compensating controls |
| 3.2 | Apply change management | P0/P1 = Emergency Change; all production changes require rollback plan |
| 3.3 | Patch testing | Never skip staging; compress timeline for P0 but always smoke-test |
| 3.4 | Special scenarios | Third-party, open source dependencies, cloud IaC each have distinct workflows |

**Remediation Strategies:**

| Strategy | When to Use | Permanence |
|---|---|---|
| Patch / Update | Vendor patch available | Permanent |
| Configuration Hardening | Misconfiguration root cause | Permanent |
| Compensating Control | Patch not yet available, business constraint | Temporary (time-bounded) |
| Risk Acceptance | Cost/impact disproportionate; EOL; decommission scheduled | Temporary (max 90 days Critical) |
| Decommission | No business purpose; unmitigable risk | Permanent |

#### Phase 4: REASSESS — 4 Steps

| Step | Action | Key Recommendation |
|---|---|---|
| 4.1 | Execute validation scans | Run targeted re-scan immediately post-remediation; P0/P1 within 24 hours |
| 4.2 | Collect remediation evidence | Scan results + version screenshots + config before/after; for Microsoft patches: confirm KB article number via MSRC advisory + `Get-HotFix`; store in VMP not email |
| 4.3 | Handle partial and recurring findings | Recurring findings = root cause not addressed; investigate regression |
| 4.4 | Update downstream systems | CMDB, ticketing, VMP, compliance evidence repository, stakeholder notification |

#### Phase 5: IMPROVE — 5 Steps

| Step | Action | Key Recommendation |
|---|---|---|
| 5.1 | Root cause analysis (RCA) | Mandatory for P0/P1 and recurring patterns; identify process, dev, or architecture gap |
| 5.2 | Update policies and standards | Match updates to recurring finding patterns |
| 5.3 | Implement shift-left controls | Embed SAST, SCA, secrets detection, and IaC scanning into every CI/CD pipeline |
| 5.4 | Track and improve KPIs | MTTD, MTTR, SLA compliance, KEV coverage rate, vulnerability density; monitor MSRC Blog + RSS for emerging Microsoft CVEs |
| 5.5 | Conduct tabletop and pentest | Annual minimum; test escalation paths and zero-day response |

**Core KPIs:**

| KPI | Target |
|---|---|
| Scan Coverage Rate (Tier 1–2) | > 95% |
| MTTD (Critical) | < 7 days |
| SLA Compliance Rate | > 90% |
| Recurrence Rate | < 5% |
| KEV Coverage Rate | 100% |

#### Phase 6: REPORT — 5 Steps

| Step | Action | Key Recommendation |
|---|---|---|
| 6.1 | Define audiences and needs | Board = business risk; DevTeam = SARIF in CI/CD; Audit = PDF + scan exports |
| 6.2 | Build executive dashboard | Open Critical/High counts, trend, MTTR, KEV status, SLA compliance |
| 6.3 | Produce technical reports | Every finding: CVE, CWE, CVSS vector, EPSS, KEV status, owner, due date; include MSRC Severity Rating + Exploitability Index + advisory URL for all Microsoft CVEs |
| 6.4 | Select output format | SARIF for pipelines; CycloneDX VEX for SBOM; STIX for threat intel; PDF/CSV for audit |
| 6.5 | Align to compliance frameworks | Map every report to applicable control evidence requirement; for Microsoft environments include WSUS/MECM deployment reports + MSRC advisory links as evidence |

**Compliance Evidence Requirements:**

| Framework | Key Requirement | Evidence |
|---|---|---|
| PCI DSS v4.0 | Req 11.3 — Quarterly scans | Passing scan reports, ASV report |
| HIPAA | 45 CFR §164.308(a)(8) | Scan + remediation records |
| SOC 2 | CC7.1 — Threat detection | Continuous scan evidence, SIEM logs |
| ISO 27001:2022 | A.8.8 — Technical vuln management | Policy + records across all phases |
| FedRAMP | RA-5, CA-7 — Monthly scanning | Monthly scan reports, POA&M |
| CIS Controls v8 | Control 7 — Continuous management | Coverage rate, MTTR, SLA metrics |
| MSRC / Microsoft environments | Patch Tuesday compliance | WSUS/MECM/Intune deployment reports + MSRC advisory links + KB article confirmation |

**Official Reference:** NIST SP 800-40 Rev 4 — https://csrc.nist.gov/publications/detail/sp/800-40/rev-4/final

---

### 4. Vendor Tool Analysis

**Reference file:** `vendor-tool-comparison.md`

When a vulnerability question is asked, the agent evaluates coverage across all 6 tools **in this fixed order**, providing plugin IDs, detection logic, and remediation guidance for each:

#### Tool Priority Order

```
1. Tenable (Nessus / Tenable.io / Tenable.sc)
2. OpenVAS (Greenbone Vulnerability Management)
3. Nexpose / InsightVM (Rapid7)
4. Retina CS (BeyondTrust)
5. GFI LanGuard
6. Qualys FreeScan / VMDR
```

#### Comparative Summary

| Feature | Tenable | OpenVAS | Nexpose | Retina | GFI LanGuard | Qualys |
|---|---|---|---|---|---|---|
| **Plugin/Check Count** | 100,000+ | 70,000+ | 170,000+ | ~60,000 | 60,000+ | 150,000+ |
| **Free Tier** | Essentials (16 IPs) | Community (unlimited) | Community (32 IPs) | Community (32 IPs) | Trial only | FreeScan (10 IPs) |
| **Agent-Based Scanning** | Yes | No (community) | Yes | Yes | Yes | Yes |
| **Cloud Assessment** | Yes | Limited | Yes | Limited | No | Yes |
| **Container Scanning** | Yes | No | Yes | No | No | Yes |
| **Built-in Patch Management** | No | No | No | No | **Yes** | No |
| **CVSS v4.0 Support** | In progress | v2/v3 | In progress | v2/v3 | v2/v3 | **Yes** |
| **EPSS / Risk Prioritization** | VPR | No | Real Risk Score | No | No | TruRisk |
| **OT / ICS Support** | Tenable.ot | No | No | No | No | No |
| **Compliance Audits** | Extensive | BSI, PCI | PCI, HIPAA | PCI, HIPAA | PCI, CIS | Extensive |
| **SARIF Export** | Via API | Community tools | Via API | Limited | Limited | WAS module |

#### Per-Tool Analysis for Each Query Includes

- Does the tool detect this CVE/CWE?
- What plugin/QID/check ID covers it?
- How quickly was coverage added after CVE publication?
- What remediation guidance does the tool provide?
- What output/report format does it produce?
- Licensing model (free vs. commercial)

#### Official Documentation Links

| Tool | Docs | Plugin/Check Search |
|---|---|---|
| Tenable | https://docs.tenable.com | https://www.tenable.com/plugins |
| OpenVAS | https://greenbone.github.io/docs/ | https://community.greenbone.net |
| Nexpose / InsightVM | https://docs.rapid7.com/insightvm/ | https://www.rapid7.com/db/ |
| Retina CS | https://www.beyondtrust.com/docs/retina/ | BeyondTrust console |
| GFI LanGuard | https://www.gfi.com/products-and-solutions/network-security-solutions/gfi-languard | GFI console |
| Qualys | https://docs.qualys.com | https://www.qualys.com/research/security-advisories/ |

---

### 5. Output Standardization

**Reference file:** `output-standardization.md`

The agent recommends the appropriate format for centralizing and sharing vulnerability results, and provides converter references for each scanner.

#### Supported Output Standards

| Standard | Use Case | Official Reference |
|---|---|---|
| **SARIF v2.1** | SAST/DAST/scan results, CI/CD pipelines, GitHub/Azure DevOps | https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html |
| **CycloneDX** | SBOM + VEX (Vulnerability Exploitability eXchange) for software supply chain | https://cyclonedx.org |
| **SCAP** | NIST Security Content Automation (OVAL, XCCDF, CPE) | https://scap.nist.gov |
| **STIX/TAXII** | Threat intelligence sharing between organizations | https://oasis-open.github.io/cti-documentation/ |
| **Nessus XML (.nessus)** | Native Tenable format, widely supported by parsers | https://docs.tenable.com |

#### Scanner → SARIF Converters

| Scanner | Converter | Link |
|---|---|---|
| Nessus (.nessus) | nessus-sarif-converter | https://github.com/Stift007/nessus-sarif-converter |
| OpenVAS (XML) | openvas-sarif | https://github.com/ShantoNoor/OpenVAS-to-SARIF |
| Bandit (Python SAST) | Built-in | `bandit -r . -f sarif -o results.sarif` |
| Semgrep | Built-in | `semgrep --sarif --output=results.sarif` |
| Trivy (containers) | Built-in | `trivy image --format sarif myimage` |
| Checkov (IaC) | Built-in | `checkov -d . --output sarif` |
| OWASP ZAP | Built-in | ZAP → Report → SARIF |
| Grype | Built-in | `grype myimage --output sarif` |

#### Centralization Architecture

```
SCANNER LAYER
├── Tenable (.nessus XML)
├── OpenVAS (XML)
├── Nexpose (XML)
└── Qualys (XML)
        │
        ▼
FORMAT CONVERSION LAYER
(SARIF / CycloneDX converters)
        │
        ▼
CENTRALIZATION PLATFORM
DefectDojo / Dependency-Track
        │
        ├── SIEM (Microsoft Sentinel, Splunk, QRadar)
        ├── Ticketing (Jira, ServiceNow)
        └── Dashboards (Power BI, Grafana)
```

#### Recommended Centralization Platform

**DefectDojo** (open source) — https://defectdojo.com

Supports 150+ scanner import formats, deduplication, Jira/ServiceNow integration, SARIF/CycloneDX export, and REST API for CI/CD pipelines.

```bash
git clone https://github.com/DefectDojo/django-DefectDojo
cd django-DefectDojo
./dc-up.sh
# Access at http://localhost:8080
```

---

### 6. AI-Powered Vulnerability Assessment Tools

**Reference file:** `ai-powered-tools.md`

Covers 13 AI-powered tools available in the market, how each maps to every phase of the VMLC, and how **Microsoft Defender** serves as the centralized aggregation platform.

#### AI Tools Covered

| # | Tool | AI Type | Primary VMLC Phases |
|---|---|---|---|
| 1 | **Microsoft Security Copilot** | LLM (GPT-4o) | All phases — natural language investigation and remediation |
| 2 | **Microsoft Defender VM (MDVM)** | ML + threat intel | Discover, Prioritize, Act, Reassess |
| 3 | **Microsoft Defender for Cloud** | ML + graph AI | Discover, Prioritize, Act, Report |
| 4 | **Microsoft Sentinel** | ML (Fusion AI) | Prioritize, Reassess, Improve, Report |
| 5 | **GitHub Copilot Autofix** | LLM | Discover, Act, Reassess |
| 6 | **Snyk (DeepCode AI)** | LLM + reachability ML | Discover, Prioritize, Act |
| 7 | **Tenable ExposureAI** | ML + GenAI | Discover, Prioritize, Improve, Report |
| 8 | **Qualys TotalCloud** | ML + GenAI | Discover, Prioritize, Act, Report |
| 9 | **Wiz** | Graph AI | Discover, Prioritize, Act, Reassess |
| 10 | **Orca Security** | ML (agentless) | Discover, Prioritize, Reassess |
| 11 | **Rapid7 InsightVM** | ML | Discover, Prioritize, Act |
| 12 | **CrowdStrike Falcon** | ML + threat intel | Discover, Prioritize, Reassess |
| 13 | **Darktrace** | Unsupervised ML | Prioritize, Act, Reassess |

#### Microsoft Defender as Central Aggregation Hub

Microsoft Defender unifies all vulnerability signals from native Microsoft tools and third-party scanners into a single platform:

```
Tenable / Qualys / Rapid7 / Wiz / Orca / CrowdStrike / Darktrace
    └── Microsoft Sentinel (Data Connectors)
            └── Defender XDR Portal (Unified Dashboard)
                    └── Security Copilot (AI Reasoning + Reports)

GitHub GHAS / Snyk / Azure DevOps
    └── Defender for DevOps
            └── Defender for Cloud (Unified Posture)
                    └── Security Copilot
```

| Defender Component | Role |
|---|---|
| **Defender XDR Portal** | Unified investigation and vulnerability dashboard |
| **Defender Vulnerability Management** | Endpoint CVE tracking, exposure score, MSRC integration |
| **Defender for Cloud** | Cloud posture, attack path analysis, IaC scanning |
| **Microsoft Sentinel** | Third-party tool aggregation, KQL hunting, SOAR playbooks |
| **Security Copilot** | AI reasoning layer — natural language queries, remediation scripts, executive reports |
| **Defender for DevOps** | DevSecOps findings from GitHub, Azure DevOps, Snyk |

---

### 7. Staying Current

The agent is designed to reference live update sources for newly published CVEs, CWEs, and CVSS changes, and to evaluate the impact on each supported scanner.

#### Live Update Sources

| Source | URL | Update Frequency |
|---|---|---|
| NVD CVE Feed | https://nvd.nist.gov/vuln/search | Real-time |
| NVD API | https://nvd.nist.gov/developers/vulnerabilities | Real-time |
| CVE JSON Feed (official) | https://github.com/CVEProject/cvelistV5 | Daily |
| CISA KEV Catalog | https://www.cisa.gov/known-exploited-vulnerabilities-catalog | Daily |
| CISA KEV JSON | https://www.cisa.gov/sites/default/files/feeds/known_exploited_vulnerabilities.json | Daily |
| MITRE CWE News | https://cwe.mitre.org/news/ | Per release |
| FIRST CVSS Updates | https://www.first.org/cvss/ | Per version |
| EPSS API | https://api.first.org/data/v1/epss | Daily |
| Tenable Plugin Feed | https://www.tenable.com/plugins | Multiple/day |
| Greenbone NVT Community Feed | https://community.greenbone.net | Daily |
| Rapid7 VulnDB | https://www.rapid7.com/db/ | Daily |
| Qualys Security Advisories | https://www.qualys.com/research/security-advisories/ | Daily |

#### When a New CVE/CWE/CVSS Is Published, the Agent Evaluates

1. Does it affect software in the user's stack?
2. Which scanner vendor published coverage first?
3. Is it in CISA KEV? (confirmed active exploitation)
4. What is the EPSS score? (exploitation probability in 30 days)
5. What is the CVSS v3.1 and v4.0 score and vector?
6. What compensating controls apply while patches are pending?
7. What is the CWE root cause, and does it appear in the CWE Top 25?

---

### 8. Defender Alert Analysis

**Reference file:** `defender-alert-analysis.md`

Arcus analyzes Microsoft Defender alerts end-to-end. Given a raw alert (JSON paste, alert ID, or plain-language description), it produces a 10-section response:

| Section | Content |
|---|---|
| **1. Alert Summary** | Title, ID, severity, status, affected asset, detection timestamp |
| **2. Plain-Language Explanation** | What happened, what was targeted, what the attacker was attempting |
| **3. MITRE ATT&CK Mapping** | Tactic → Technique → Sub-technique; kill chain position |
| **4. Composite Severity Score (CSS)** | Weighted formula → P0–P4 priority tier |
| **5. Asset Context** | Criticality, exposure level, blast radius |
| **6. Threat Actor Context** | Known ATT&CK Groups using this technique |
| **7. Recommended Response Actions** | Graph API actions: isolate, disable, block, collect |
| **8. KQL Hunting Query** | Sentinel query to hunt for related activity |
| **9. VMLC Phase Mapping** | Lifecycle phase (typically DISCOVER or PRIORITIZE) |
| **10. References** | Graph API docs, MITRE ATT&CK entry, Microsoft Learn |

#### Composite Severity Score (CSS)

CSS overrides raw Defender severity by incorporating 5 weighted factors:

| Factor | Weight | Input Range |
|---|---|---|
| Defender Severity | 30% | Informational=10, Low=40, Medium=60, High=80, Critical=100 |
| MITRE Tactic Weight | 25% | Impact=100 → Reconnaissance=30 |
| Asset Criticality | 20% | DC/CA=100, Server=75, Workstation=50, Non-managed=25 |
| Active Exploitation Signal | 15% | KEV/MSRC exploited=100, PoC=75, Theoretical=25, None=0 |
| Blast Radius | 10% | Domain-wide=100, Subnet=75, Host=50, Isolated=25 |

**Priority Tiers:**

| CSS Range | Priority | SLA |
|---|---|---|
| 80–100 | P0 — Critical | < 1 hour |
| 60–79 | P1 — High | < 4 hours |
| 40–59 | P2 — Medium | < 24 hours |
| 20–39 | P3 — Low | < 72 hours |
| 0–19 | P4 — Informational | Next business day |

> Domain Controllers, executive accounts, and internet-facing assets escalate one tier regardless of CSS.

**Official References:**
- Microsoft Security Graph API: https://learn.microsoft.com/en-us/graph/api/resources/security-api-overview
- MITRE ATT&CK: https://attack.mitre.org
- Microsoft Sentinel KQL: https://learn.microsoft.com/en-us/azure/sentinel/kusto-overview

---

## How to Use the Agent

### Example Questions

```
"How do I deal with Python weaknesses in my application?"
"What is the risk of CVE-2021-44228 (Log4Shell) in my environment?"
"How does Tenable detect CWE-89 (SQL Injection)?"
"What tools detect default credentials on network devices?"
"How do I export Nessus results to SARIF for GitHub?"
"What phase of the vulnerability management lifecycle is patching?"
"How do I prioritize 500 vulnerabilities found in a scan?"
"What new CVEs were published this week that affect Linux?"
```

### Response Format (Every Answer)

Every Arcus response follows this structure:

```
1. CLASSIFICATION    — Which of the 8 vulnerability categories applies
2. CWE MAPPING       — Root cause weakness type(s)
3. CVE + CVSS        — Specific CVE(s) with v3.1/v4.0 score and vector string
4. KEV STATUS        — Is this in the CISA Known Exploited Vulnerabilities catalog?
5. TOOL COVERAGE     — Tenable → OpenVAS → Nexpose → Retina → GFI LanGuard → Qualys
6. LIFECYCLE PHASE   — Where in Discover/Prioritize/Act/Reassess/Improve/Report
7. OUTPUT FORMAT     — SARIF, CycloneDX, or other recommended standard
8. REFERENCES        — Official links (NVD, MITRE, FIRST, NIST, vendor docs)
```

---

## Official References

| Resource | URL |
|---|---|
| NVD — National Vulnerability Database | https://nvd.nist.gov |
| MITRE CVE | https://cve.mitre.org |
| MITRE CWE | https://cwe.mitre.org |
| CWE Top 25 (2024) | https://cwe.mitre.org/top25/ |
| FIRST CVSS | https://www.first.org/cvss/ |
| FIRST CVSS v4.0 | https://www.first.org/cvss/v4.0/specification-document |
| FIRST EPSS | https://www.first.org/epss/ |
| CISA KEV Catalog | https://www.cisa.gov/known-exploited-vulnerabilities-catalog |
| OWASP Top 10 | https://owasp.org/Top10/ |
| OWASP API Security Top 10 | https://owasp.org/API-Security/ |
| OWASP SAMM | https://owaspsamm.org/ |
| CIS Controls v8 | https://www.cisecurity.org/controls/v8 |
| CIS Benchmarks | https://www.cisecurity.org/cis-benchmarks |
| NIST SP 800-30 Rev 1 (Risk Assessment) | https://csrc.nist.gov/publications/detail/sp/800-30/rev-1/final |
| NIST SP 800-40 Rev 4 (Patch Management) | https://csrc.nist.gov/publications/detail/sp/800-40/rev-4/final |
| NIST SP 800-115 (Security Testing) | https://csrc.nist.gov/publications/detail/sp/800-115/final |
| NIST SP 800-160 Vol 1 (Security Engineering) | https://csrc.nist.gov/publications/detail/sp/800-160/vol-1/rev-1/final |
| NIST Cybersecurity Framework 2.0 | https://www.nist.gov/cyberframework |
| SARIF v2.1 Specification | https://docs.oasis-open.org/sarif/sarif/v2.1.0/sarif-v2.1.0.html |
| SARIF Validator | https://sarifweb.azurewebsites.net |
| CycloneDX | https://cyclonedx.org |
| CycloneDX VEX | https://cyclonedx.org/capabilities/vex/ |
| SCAP | https://scap.nist.gov |
| OVAL | https://oval.mitre.org |
| OpenSCAP | https://www.open-scap.org |
| STIX / TAXII | https://oasis-open.github.io/cti-documentation/ |
| MITRE ATT&CK | https://attack.mitre.org |
| DefectDojo | https://defectdojo.com |
| OWASP Dependency-Track | https://dependencytrack.org |
| OSV (Open Source Vulnerabilities) | https://osv.dev |
| GitHub Advisory Database | https://github.com/advisories |
| Tenable Docs | https://docs.tenable.com |
| Tenable Plugin Feed | https://www.tenable.com/plugins |
| Greenbone / OpenVAS Docs | https://greenbone.github.io/docs/ |
| Rapid7 InsightVM Docs | https://docs.rapid7.com/insightvm/ |
| Rapid7 Vulnerability Database | https://www.rapid7.com/db/ |
| BeyondTrust Retina Docs | https://www.beyondtrust.com/docs/retina/ |
| GFI LanGuard | https://www.gfi.com/products-and-solutions/network-security-solutions/gfi-languard |
| Qualys Docs | https://docs.qualys.com |
| Qualys Security Advisories | https://www.qualys.com/research/security-advisories/ |
| PCI DSS v4.0 | https://www.pcisecuritystandards.org/ |
| HIPAA | https://www.hhs.gov/hipaa/ |
| ISO/IEC 27001:2022 | https://www.iso.org/standard/82875.html |
| FedRAMP | https://www.fedramp.gov/ |
| DISA STIGs | https://public.cyber.mil/stigs/ |
| **Microsoft Security Copilot** | https://learn.microsoft.com/en-us/security-copilot/ |
| **Microsoft Defender Vulnerability Management** | https://learn.microsoft.com/en-us/defender-vulnerability-management/ |
| **Microsoft Defender for Cloud** | https://learn.microsoft.com/en-us/azure/defender-for-cloud/ |
| **Microsoft Defender for Cloud — Attack Path Analysis** | https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-attack-path |
| **Microsoft Sentinel** | https://learn.microsoft.com/en-us/azure/sentinel/ |
| **Defender for DevOps** | https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-devops-introduction |
| **GitHub Copilot Autofix (GHAS)** | https://docs.github.com/en/code-security/code-scanning/managing-code-scanning-alerts/about-autofix-for-codeql-alerts |
| **Snyk DeepCode AI** | https://snyk.io/product/snyk-code/ |
| **Tenable ExposureAI / Tenable One** | https://www.tenable.com/products/tenable-one/exposure-ai |
| **Qualys TotalCloud** | https://www.qualys.com/apps/totalcloud/ |
| **Wiz CNAPP** | https://www.wiz.io/platform/wiz-cnapp |
| **Orca Security** | https://orca.security/platform/ |
| **CrowdStrike Falcon Exposure Management** | https://www.crowdstrike.com/platform/exposure-management/ |
| **Darktrace PREVENT** | https://darktrace.com/products/prevent/ |
| **MSRC Security Update Guide** | https://msrc.microsoft.com/update-guide/ |
| **MSRC Blog** | https://msrc.microsoft.com/blog/ |
| **Microsoft Security Graph API** | https://learn.microsoft.com/en-us/graph/api/resources/security-api-overview |
| **Microsoft Defender Portal** | https://security.microsoft.com |
| **Microsoft Sentinel KQL Reference** | https://learn.microsoft.com/en-us/azure/sentinel/kusto-overview |
| **MITRE ATT&CK TAXII / Data Tools** | https://attack.mitre.org/resources/attack-data-and-tools/ |
| **MITRE ATT&CK Groups** | https://attack.mitre.org/groups/ |
