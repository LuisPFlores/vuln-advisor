# Vulnerability Scanner Vendor Comparison

## Overview

This reference compares the six primary vulnerability scanning platforms used in the industry. When evaluating tool coverage for a specific vulnerability, consult them **in this priority order**:

1. Tenable (Nessus / Tenable.io / Tenable.sc)
2. OpenVAS (Greenbone Vulnerability Management)
3. Nexpose / InsightVM (Rapid7)
4. Retina CS (BeyondTrust)
5. GFI LanGuard
6. Qualys FreeScan / VMDR

---

## 1. Tenable — Nessus / Tenable.io / Tenable.sc

**Vendor:** Tenable Holdings, Inc.
**Official Docs:** https://docs.tenable.com
**Plugin Database:** https://www.tenable.com/plugins
**Community:** https://community.tenable.com

### Products

| Product | Target | Model |
|---|---|---|
| Nessus Essentials | Individuals / Students (up to 16 IPs) | Free |
| Nessus Professional | Pentesters / SMBs | Commercial |
| Tenable.io (VMDR) | Enterprise cloud-based | SaaS |
| Tenable.sc | Enterprise on-premises | Commercial |
| Tenable.ot | OT/ICS environments | Commercial |
| Tenable.cs | Cloud Security Posture | SaaS |

### Key Capabilities
- **Plugin count:** 100,000+ plugins (largest in industry)
- **Plugin families:** CGI abuses, Windows, Linux, Web Servers, Databases, Network devices, Cloud, Compliance
- **Scan types:** Network scan, web application scan, agent scan, cloud audit
- **Credentialed scanning:** SSH, WMI/SMB, SNMP, database credentials
- **Compliance audits:** CIS, DISA STIG, PCI-DSS, HIPAA, NIST
- **CVSS support:** v2, v3, v4 (Tenable Risk Score also available)
- **CVE/CWE mapping:** Every plugin maps to CVE(s) and CWE where applicable

### Finding a Plugin for a Specific CVE/CWE

```
# Search by CVE:
https://www.tenable.com/plugins/search?q=CVE-2021-44228

# Search by CWE:
https://www.tenable.com/plugins/search?q=CWE-79

# Plugin output includes:
- Plugin ID, Name, Family
- CVE(s) covered
- CVSS v3 vector and score
- Solution / remediation steps
- References (vendor advisories, NVD)
```

### Output Formats
- `.nessus` (XML) — native format
- CSV, HTML, PDF exports
- Syslog / SIEM integration
- REST API: https://developer.tenable.com/

### Tenable Lumin / Risk-Based Vulnerability Management
- Asset Criticality Rating (ACR) × Vulnerability Priority Rating (VPR)
- VPR considers CVSS + threat intelligence + EPSS-like scoring
- **Reference:** https://docs.tenable.com/tenableio/Content/Settings/VPR.htm

---

## 2. OpenVAS — Greenbone Vulnerability Management (GVM)

**Vendor:** Greenbone Networks GmbH
**Official Docs:** https://greenbone.github.io/docs/
**Community Feed:** https://community.greenbone.net/
**Source Code:** https://github.com/greenbone

### Products

| Product | Target | Model |
|---|---|---|
| Greenbone Community Edition | Individuals / SMBs | Open Source (free) |
| Greenbone Enterprise Appliance | Enterprise | Commercial hardware/VM |
| Greenbone Cloud Service | Enterprise | SaaS |

### Key Capabilities
- **NVT (Network Vulnerability Tests):** 70,000+ tests
- **Feed types:** Greenbone Community Feed (free) vs. Greenbone Enterprise Feed (commercial, updated more frequently)
- **Scan types:** Full scan, discovery scan, host scan, web application scan
- **GMP API:** Greenbone Management Protocol for automation
- **Credentialed scanning:** SSH, SMB, ESXi, SNMP
- **Compliance scanning:** BSI IT-Grundschutz, PCI-DSS
- **CVSS support:** v2, v3

### Architecture

```
GSA (Greenbone Security Assistant — Web UI)
  └── GVM (Greenbone Vulnerability Manager — daemon)
        └── OpenVAS Scanner (scanning engine)
              └── NVT Feed (vulnerability tests)
        └── PostgreSQL Database
        └── GMP API (automation)
```

### Finding NVTs for Specific CVEs

```
# Via Web UI: Scans → Results → Filter by CVE
# Via GMP CLI:
gvm-cli socket --xml "<get_nvts/>" | grep "CVE-2021-44228"

# NVT families include:
- Web application abuses
- Buffer overflow
- Default Accounts
- Denial of Service
- Gain a shell remotely
```

### Output Formats
- XML reports
- PDF, HTML, CSV, TXT exports
- GMP API for integration
- **OpenVAS to SARIF:** Community converters available
  - https://github.com/ShantoNoor/OpenVAS-to-SARIF

### Strengths
- Free, open-source (Community Edition)
- Extensive NVT library
- Good for budget-conscious environments
- Docker deployment available: https://greenbone.github.io/docs/latest/22.4/container/

---

## 3. Nexpose / InsightVM — Rapid7

**Vendor:** Rapid7
**Official Docs:** https://docs.rapid7.com/insightvm/
**Plugin/Check Database:** https://www.rapid7.com/db/
**Community:** https://discuss.rapid7.com/c/insightvm

### Products

| Product | Target | Model |
|---|---|---|
| Nexpose Community | Up to 32 IPs | Free |
| Nexpose Express | SMBs | Commercial |
| InsightVM | Enterprise | SaaS (cloud-connected) |
| tCell | Runtime app security | SaaS add-on |

### Key Capabilities
- **Check count:** 170,000+ vulnerability checks
- **Real Asset Risk Score:** Risk-prioritized scoring beyond CVSS
- **Live monitoring:** InsightVM agents for real-time assessment
- **Container security:** Docker/Kubernetes scanning
- **Cloud assessment:** AWS, Azure, GCP configuration checks
- **Remediation projects:** Assign, track, and measure remediation
- **CVSS support:** v2, v3 (working on v4 integration)

### Real Risk Score

Rapid7 Real Risk Score (0–1000) factors in:
- CVSS base score
- Temporal score (exploit availability)
- Asset criticality
- Network exposure

**Reference:** https://docs.rapid7.com/insightvm/real-risk/

### Nexpose Check Categories
```
Categories include:
- Remote Exploit
- Local Exploit
- Denial of Service
- Authentication Bypass
- Information Disclosure
- Configuration Audit
- Policy Compliance
```

### Finding Checks for CVEs

```
# Via Rapid7 Vulnerability Database:
https://www.rapid7.com/db/?q=CVE-2021-44228&type=nexpose

# Each check entry includes:
- Nexpose Check ID
- CVE(s) mapped
- CVSS score
- Vulnerability proof (how it's detected)
- Solution / remediation
```

### Output Formats
- XML, CSV, PDF, HTML reports
- REST API: https://help.rapid7.com/insightvm/en-us/api/
- SIEM integration (Splunk, QRadar, Splunk SOAR)
- DefectDojo integration

### Unique Features
- **Exposure Analytics:** Visualize attack paths
- **Remediation Projects:** Track fixes by team/ticket
- **Integration with Metasploit:** Validate exploitability
  - https://docs.rapid7.com/insightvm/metasploit-integration/

---

## 4. Retina CS — BeyondTrust

**Vendor:** BeyondTrust (formerly eEye Digital Security)
**Official Docs:** https://www.beyondtrust.com/docs/retina/
**Product Page:** https://www.beyondtrust.com/products/retina-cs-enterprise-vulnerability-management

### Products

| Product | Target | Model |
|---|---|---|
| Retina Network Security Scanner | Network scanning | Commercial |
| Retina CS (Community) | Up to 32 IPs | Free (limited) |
| Retina CS Enterprise | Enterprise | Commercial |

### Key Capabilities
- **Check database:** BeyondTrust proprietary vulnerability database
- **Smart scanning:** Adaptive scanning that learns network topology
- **Integration with BeyondTrust PAM:** Privileged Access Management integration for credentialed scanning
- **Agentless scanning:** No agent required
- **Compliance scanning:** PCI-DSS, HIPAA, CIS
- **CVSS support:** v2, v3

### Notable Features
- **Zero-Day Protection:** BeyondTrust Retina has historically been known for rapid detection of new vulnerabilities
- **OVAL support:** Open Vulnerability and Assessment Language
  - https://oval.mitre.org/
- **Database security scanning:** Oracle, SQL Server vulnerability assessment

### Finding Retina Checks

```
# Retina updates vulnerability database via LiveUpdate
# Check search available in Retina CS console
# Audits mapped to CVE, OVAL IDs, and BeyondTrust check IDs
```

### Output Formats
- XML, PDF, CSV, HTML reports
- OVAL output support
- Integration with BeyondTrust PowerBroker

**Note:** Retina CS has seen reduced market activity compared to Tenable and Rapid7. BeyondTrust has pivoted focus toward PAM solutions. Verify current product availability at https://www.beyondtrust.com.

---

## 5. GFI LanGuard

**Vendor:** GFI Software
**Official Docs:** https://www.gfi.com/products-and-solutions/network-security-solutions/gfi-languard/documentation
**Product Page:** https://www.gfi.com/products-and-solutions/network-security-solutions/gfi-languard

### Products

| Product | Target | Model |
|---|---|---|
| GFI LanGuard | SMBs, MSPs | Commercial (per node) |

### Key Capabilities
- **Check count:** 60,000+ vulnerability checks
- **Patch management built-in:** Deploys patches directly from the console
- **Software audit:** Detects unauthorized software
- **Asset discovery:** Network and hardware inventory
- **OVAL/CVSS integration:** Uses OVAL definitions and CVSS scoring
- **Platforms:** Windows, Linux, macOS, network devices
- **Agents:** Optional GFI LanGuard agent for always-on scanning

### Patch Management Focus

GFI LanGuard is unique in combining vulnerability scanning **with** patch deployment:

```
Scan → Identify Missing Patches → Deploy Patches
         └── Microsoft (Windows Update)
         └── Third-party (Adobe, Chrome, Java, etc.)
         └── Linux (apt/yum)
```

**This makes it particularly strong for "Poor Patch Management" vulnerability class.**

### Finding Checks for CVEs

```
# GFI LanGuard maps checks to:
- Microsoft KB articles
- CVE identifiers
- OVAL definitions
# Search via GFI LanGuard console: Results → Filter by CVE
```

### Output Formats
- HTML, PDF, XML reports
- Dashboard with trend analysis
- Syslog export
- API for integration

### Best Use Cases
- SMBs needing scan + patch in one tool
- MSPs managing multiple client environments
- Organizations focused on Microsoft patch compliance

---

## 6. Qualys FreeScan / VMDR

**Vendor:** Qualys, Inc.
**Official Docs:** https://docs.qualys.com
**QID Database:** https://www.qualys.com/research/security-advisories/
**Community:** https://community.qualys.com

### Products

| Product | Target | Model |
|---|---|---|
| Qualys FreeScan | Up to 10 IPs (external only) | Free |
| Qualys VMDR | Enterprise | SaaS |
| Qualys WAS | Web Application Scanning | SaaS |
| Qualys CS | Container Security | SaaS |
| Qualys CSPM | Cloud Security Posture | SaaS |
| Qualys CertView | SSL/TLS Certificate Visibility | SaaS |
| Qualys TotalCloud | Multi-cloud security | SaaS |

### Key Capabilities
- **QID (Qualys ID):** Proprietary vulnerability identifier
- **Knowledgebase:** Qualys Security Knowledge Base with 150,000+ QIDs
- **Cloud agents:** Lightweight agents for always-on scanning
- **TruRisk Score:** Risk-prioritized scoring beyond CVSS
  - Combines CVSS, asset criticality, threat intelligence
- **Container/IaC scanning:** Integrated with CI/CD
- **Compliance:** PCI-DSS, HIPAA, SOC2, CIS, NIST scanning
- **CVSS support:** v2, v3, v4 (Qualys has been an early adopter)

### Qualys TruRisk

TruRisk factors in:
- CVSS base score
- Qualys Detection Score (QDS) — exploit availability, malware association
- Asset business criticality
- Exposure (internet-facing vs. internal)

**Reference:** https://www.qualys.com/solutions/trurisk/

### Finding QIDs for CVEs

```
# Qualys KnowledgeBase search:
https://www.qualys.com/research/security-advisories/

# In VMDR console: KnowledgeBase → Search by CVE
# Each QID includes:
- QID number
- Vulnerability title
- CVE(s) mapped
- CVSS vector and score
- Threat intelligence indicators
- Detection logic
- Remediation steps
```

### Output Formats
- XML, CSV, PDF, HTML reports
- REST API: https://docs.qualys.com/en/vm/api/
- Connector integrations: ServiceNow, Jira, Splunk, Sentinel
- SARIF export available for Qualys WAS

### Qualys FreeScan Limitations
- External scans only (no credentialed scanning)
- Limited to 10 IP addresses
- No agent support
- Best for: Initial external attack surface assessment

---

## Comparative Summary

| Feature | Tenable | OpenVAS | Nexpose | Retina | GFI LanGuard | Qualys |
|---|---|---|---|---|---|---|
| **Checks/Plugins** | 100,000+ | 70,000+ | 170,000+ | ~60,000 | 60,000+ | 150,000+ |
| **Free Tier** | Essentials (16 IPs) | Community (unlimited) | Community (32 IPs) | Community (32 IPs) | Trial only | FreeScan (10 IPs) |
| **Agent-Based** | Yes | No (community) | Yes | Yes | Yes | Yes |
| **Cloud Scanning** | Yes | Limited | Yes | Limited | No | Yes |
| **Container Scanning** | Yes | No | Yes | No | No | Yes |
| **Built-in Patch Mgmt** | No | No | No | No | **Yes** | No |
| **CVSS v4 Support** | In progress | v2/v3 | In progress | v2/v3 | v2/v3 | **Yes** |
| **EPSS Integration** | VPR (similar) | No | Real Risk | No | No | TruRisk |
| **SARIF Export** | Via API | Community tools | Via API | Limited | Limited | WAS module |
| **Compliance Audits** | Extensive (CIS, STIG) | BSI, PCI | PCI, HIPAA | PCI, HIPAA | PCI, CIS | Extensive |
| **OT/ICS** | Tenable.ot | No | No | No | No | No |
| **Pricing Model** | Per asset/IP | Free (hardware cost) | Per asset | Per asset | Per node | Per asset |

---

## Plugin/Check Update Frequency

Rapid response to new CVEs is critical. Here is how quickly each vendor typically issues coverage:

| Vendor | Typical Plugin Release | Feed Update Frequency |
|---|---|---|
| Tenable | Hours to 24h for critical | Multiple times per day |
| Qualys | Hours to 24h for critical | Daily |
| Rapid7 (InsightVM) | 24–48h for critical | Daily |
| OpenVAS (Enterprise Feed) | 24–48h for critical | Daily |
| OpenVAS (Community Feed) | Days to weeks | Periodic |
| GFI LanGuard | 24–72h | Daily |
| Retina | Varies (verify current) | Periodic |

**For zero-day CVEs, always check:**
- Tenable: https://www.tenable.com/blog (Security Response team advisories)
- Rapid7: https://www.rapid7.com/blog/tag/vulnerability-management/
- Qualys: https://www.qualys.com/research/security-advisories/
- Greenbone: https://community.greenbone.net/

---

## References

- Tenable Documentation: https://docs.tenable.com
- Tenable Plugin Feed: https://www.tenable.com/plugins
- Greenbone/OpenVAS Docs: https://greenbone.github.io/docs/
- Greenbone GitHub: https://github.com/greenbone
- Rapid7 InsightVM Docs: https://docs.rapid7.com/insightvm/
- Rapid7 Vulnerability DB: https://www.rapid7.com/db/
- BeyondTrust Retina: https://www.beyondtrust.com/docs/retina/
- GFI LanGuard: https://www.gfi.com/products-and-solutions/network-security-solutions/gfi-languard
- Qualys Docs: https://docs.qualys.com
- Qualys Security Advisories: https://www.qualys.com/research/security-advisories/
