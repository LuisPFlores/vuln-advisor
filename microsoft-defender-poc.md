# Microsoft Defender — POC Planning and Deployment Guide

## Overview

This guide enables VulnAdvisor to take a user-described scenario and recommend the right Microsoft Defender product(s), explain the business justification, define the POC scope, provide step-by-step deployment instructions, and define success criteria.

Microsoft Defender is a **unified security platform** spanning endpoint, identity, cloud, email, applications, and SIEM/SOAR. Each product can be deployed standalone or as part of the integrated **Microsoft Defender XDR** suite.

---

## Defender Product Catalog

| Product | Protects | Core Capability | License |
|---|---|---|---|
| **Defender for Endpoint (MDE)** | Windows, macOS, Linux, iOS, Android | EDR, vulnerability management, attack surface reduction | MDE P1 / P2 / M365 E3/E5 |
| **Defender Vulnerability Management (MDVM)** | Endpoints, servers | CVE discovery, exposure score, software inventory, MSRC integration | MDVM add-on or MDE P2 |
| **Defender for Identity (MDI)** | Active Directory, Entra ID | Identity threat detection, lateral movement, credential attacks | MDI standalone / M365 E5 |
| **Defender for Office 365 (MDO)** | Exchange Online, Teams, SharePoint | Phishing, BEC, malware, safe links/attachments | MDO P1/P2 / M365 E3/E5 |
| **Defender for Cloud Apps (MDCA)** | SaaS applications, cloud usage | CASB, shadow IT discovery, app governance, DLP | MDCA / M365 E5 |
| **Defender for Cloud (MDC)** | Azure, AWS, GCP, on-premises | CSPM, CWP, attack path analysis, IaC scanning | MDC Free / Defender Plans |
| **Microsoft Sentinel** | Entire organization | SIEM/SOAR, threat hunting, UEBA, playbook automation | Pay-per-GB |
| **Microsoft Security Copilot** | All Defender products | AI-powered investigation, remediation, reporting | SCU consumption-based |
| **Defender XDR (unified)** | Cross-domain (endpoint + identity + email + cloud) | Unified incident correlation, auto-investigation, auto-response | M365 E5 / Defender XDR |

---

## Scenario-to-Product Recommendation Engine

When a user describes a scenario, map it to the correct Defender product(s) using this decision matrix:

### Decision Matrix

| Scenario Keywords | Primary Product | Secondary Product |
|---|---|---|
| Endpoints being compromised, malware, ransomware, EDR, endpoint detection | **Defender for Endpoint (MDE)** | Defender XDR |
| Missing patches, CVEs on laptops/servers, software inventory, vulnerability scanning | **Defender Vulnerability Management (MDVM)** | MDE P2 |
| Active Directory attacks, pass-the-hash, Golden Ticket, lateral movement, compromised accounts | **Defender for Identity (MDI)** | Defender XDR |
| Phishing emails, BEC, malicious attachments, email security, impersonation | **Defender for Office 365 (MDO)** | Defender XDR |
| Shadow IT, unsanctioned SaaS apps, cloud app risk, data leakage to cloud apps | **Defender for Cloud Apps (MDCA)** | MDO |
| Cloud misconfigurations, Azure/AWS/GCP security posture, attack paths in cloud, container vulnerabilities | **Defender for Cloud (MDC)** | Sentinel |
| Correlation across domains, multi-stage attacks, unified SOC, threat hunting | **Microsoft Sentinel** | Defender XDR |
| Security analyst productivity, alert triage, KQL generation, remediation scripts | **Microsoft Security Copilot** | All Defender products |
| Full XDR deployment, unified incident response, automated investigation | **Defender XDR** | All Defender products |
| DevSecOps, CI/CD vulnerabilities, IaC misconfigurations, GitHub security | **Defender for Cloud + Defender for DevOps** | GitHub GHAS |
| OT/ICS security, industrial control systems | **Defender for IoT** | Sentinel |
| Zero Trust evaluation, identity risk, conditional access + endpoint compliance | **MDE + MDI + Entra ID Protection** | Defender XDR |

---

## POC Playbooks — Per Product

---

### POC 1: Microsoft Defender for Endpoint (MDE)

**Best for:** Organizations with unprotected endpoints, seeking EDR capability, replacing legacy AV, or adding vulnerability management to endpoint fleet.

#### POC Scope Recommendation
- 10–50 devices minimum (mix of Windows, macOS if applicable)
- Duration: 2–4 weeks
- Environment: Production or dedicated POC devices in a representative segment

#### Prerequisites
- Microsoft 365 tenant (trial or existing)
- MDE P2 trial license: https://www.microsoft.com/en-us/security/business/endpoint-security/microsoft-defender-endpoint
- Devices running Windows 10/11, Windows Server 2016+, or macOS 11+
- Intune or Group Policy for onboarding (or onboarding script for manual)

#### Step-by-Step Deployment

**Step 1 — Activate Trial**
1. Go to https://security.microsoft.com
2. Navigate to **Settings → Endpoints → Onboarding**
3. Activate MDE P2 trial (30-day free trial)

**Step 2 — Onboard Devices**

*Option A — Intune (recommended for managed devices):*
```
Microsoft Intune Admin Center → Endpoint security → Endpoint detection and response
→ Create policy → Windows 10/11 → MDE onboarding
```

*Option B — Group Policy:*
```
Download onboarding package from security.microsoft.com
→ Settings → Endpoints → Onboarding → Group Policy
Deploy via GPO: Computer Configuration → Administrative Templates → Windows Defender ATP
```

*Option C — Local script (fastest for POC):*
```powershell
# Download WindowsDefenderATPOnboardingPackage.zip from portal
# Extract and run as administrator:
.\WindowsDefenderATPLocalOnboardingScript.cmd
```

**Step 3 — Verify Onboarding**
```
security.microsoft.com → Assets → Devices
→ Confirm devices appear within 5–10 minutes of onboarding
```

**Step 4 — Run Simulated Attack Test**
```powershell
# Run the MDE detection test to confirm EDR is working
powershell.exe -NoExit -ExecutionPolicy Bypass -WindowStyle Hidden $ErrorActionPreference= 'silentlycontinue';(New-Object System.Net.WebClient).DownloadFile('http://127.0.0.1/1.exe', 'C:\\test-MDATP-test\\invoice.exe');Start-Process 'C:\\test-MDATP-test\\invoice.exe'
```
*Expected: Alert appears in Defender portal within 2 minutes*

**Step 5 — Enable Attack Surface Reduction (ASR) Rules**
```
security.microsoft.com → Settings → Endpoints → Attack surface reduction
→ Enable rules in Audit mode first, then Block mode after review
```

**Step 6 — Review Vulnerability Management Dashboard**
```
security.microsoft.com → Vulnerability management → Dashboard
→ Review Exposure Score, Security Recommendations, Software Inventory
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| Device onboarding success rate | > 95% of POC devices |
| Detection test alert generated | Within 5 minutes |
| Vulnerabilities discovered on POC devices | Baseline established |
| Exposure Score calculated | Visible in dashboard |
| False positive rate in first week | < 5% |

#### Official References
- MDE Documentation: https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/
- Onboarding Guide: https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/onboarding
- MDE Trial: https://www.microsoft.com/en-us/security/business/endpoint-security/microsoft-defender-endpoint

---

### POC 2: Defender Vulnerability Management (MDVM)

**Best for:** Organizations that need CVE-level visibility across their endpoint fleet, want to replace or augment a traditional scanner, or want to integrate patch prioritization with MSRC intelligence.

#### POC Scope Recommendation
- Requires MDE P2 (MDVM is built-in) or MDVM standalone add-on
- 50–200 devices for a meaningful vulnerability dataset
- Duration: 2–3 weeks

#### Prerequisites
- MDE P2 or MDVM add-on license
- Devices onboarded to MDE (POC 1 prerequisite)

#### Step-by-Step Deployment

**Step 1 — Access MDVM Dashboard**
```
security.microsoft.com → Vulnerability management → Dashboard
```

**Step 2 — Review Exposure Score and Top Recommendations**
```
Vulnerability management → Recommendations
→ Filter by: Severity = Critical, Remediation type = Software update
→ Export top 20 recommendations to CSV for stakeholder review
```

**Step 3 — Explore the Weaknesses Page (CVE-centric view)**
```
Vulnerability management → Weaknesses
→ Search for a specific CVE (e.g., CVE-2024-21413)
→ View: Affected devices, CVSS score, MSRC Severity, Exploitability Index, KB article
```

**Step 4 — Create a Remediation Activity**
```
Vulnerability management → Recommendations
→ Select a recommendation → Request remediation
→ Set: Priority, Due date, Assign to IT team
→ Integration: Creates ticket in ServiceNow or Intune remediation task
```

**Step 5 — Enable Browser Extensions and Certificate Assessment**
```
Vulnerability management → Settings
→ Enable: Browser extensions assessment
→ Enable: Digital certificates assessment
```

**Step 6 — Set Up MSRC Integration Verification**
```
Vulnerability management → Weaknesses
→ Open any Windows CVE → Click "Microsoft Security Advisory" link
→ Verify MSRC KB article number matches the recommended update
```

**Step 7 — Configure Baseline Assessment**
```
Vulnerability management → Baselines assessment
→ Select CIS Benchmark or DISA STIG profile
→ Assign to device group
→ Review compliance gap report
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| CVEs discovered across POC fleet | Full inventory established |
| Exposure Score baseline set | Score visible with trend |
| Top 10 remediation recommendations identified | Exported and assigned |
| MSRC KB linkage verified for Windows CVEs | 100% for Critical findings |
| Remediation activity created in Intune | At least 1 end-to-end workflow tested |

#### Official References
- MDVM Overview: https://learn.microsoft.com/en-us/defender-vulnerability-management/
- Dashboard Insights: https://learn.microsoft.com/en-us/defender-vulnerability-management/tvm-dashboard-insights
- Remediation Activities: https://learn.microsoft.com/en-us/defender-vulnerability-management/tvm-remediation

---

### POC 3: Microsoft Defender for Identity (MDI)

**Best for:** Organizations concerned about Active Directory attacks, insider threats, lateral movement, credential theft, or organizations that have experienced identity-based incidents.

#### POC Scope Recommendation
- All domain controllers **must** be covered (MDI sensors on all DCs)
- Optionally: AD FS servers, Entra Connect servers
- Duration: 3–4 weeks (identity baselines take time to build)

#### Prerequisites
- MDI trial license: https://www.microsoft.com/en-us/security/business/siem-and-xdr/microsoft-defender-for-identity
- Access to all Domain Controllers (to install sensors)
- Service account with read access to AD
- Port 88 (Kerberos), 389 (LDAP), 636 (LDAPS) accessible from sensors to DCs

#### Step-by-Step Deployment

**Step 1 — Create MDI Instance**
```
security.microsoft.com → Settings → Identities → Sensors
→ Download sensor setup package
```

*Or via Microsoft Defender portal:*
```
https://security.microsoft.com → Settings → Microsoft Defender for Identity
```

**Step 2 — Install Sensor on Domain Controllers**
```powershell
# Run on each Domain Controller as Administrator
.\Azure ATP Sensor Setup.exe /quiet NetFrameworkCommandLineArguments="/q" `
  AccessKey="<your-access-key-from-portal>"
```

**Step 3 — Configure Directory Service Account**
```
MDI portal → Settings → Directory Service Accounts
→ Add a gMSA or standard service account with:
  - Read access to all AD objects
  - Permission to read deleted objects
```

**Step 4 — Verify Sensor Health**
```
security.microsoft.com → Settings → Identities → Sensors
→ All DCs should show Status: Running within 10 minutes
```

**Step 5 — Simulate Identity Attacks for Detection Validation**

*Reconnaissance:*
```powershell
# LDAP reconnaissance (triggers MDI alert)
Get-ADUser -Filter * -Properties * | Select-Object Name, SamAccountName | Export-Csv users.csv
```

*Pass-the-Hash simulation (use authorized pentest tool):*
```
Use Mimikatz in lab: sekurlsa::pth /user:admin /domain:contoso /ntlm:<hash>
Expected: MDI raises "Pass-the-Hash" alert within minutes
```

**Step 6 — Review Identity Security Posture**
```
security.microsoft.com → Secure Score → Identity
→ Review identity-related recommendations
→ Defender for Identity → Reports → Sensitive accounts, Lateral movement paths
```

**Step 7 — Review Lateral Movement Paths**
```
MDI portal → Lateral movement paths
→ View graph of paths from compromised accounts to sensitive targets
→ Identify over-privileged accounts that create attack paths
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| All DCs covered with sensors | 100% |
| Sensor health status: Running | 100% |
| Reconnaissance simulation detected | Alert within 5 minutes |
| Lateral movement paths identified | At least 1 path discovered and reviewed |
| Identity Secure Score improvement recommendations | Top 5 identified and assigned |

#### Official References
- MDI Documentation: https://learn.microsoft.com/en-us/defender-for-identity/
- Sensor Installation: https://learn.microsoft.com/en-us/defender-for-identity/sensor-deployment-wizard
- Attack Simulations: https://learn.microsoft.com/en-us/defender-for-identity/playbooks

---

### POC 4: Microsoft Defender for Office 365 (MDO)

**Best for:** Organizations experiencing phishing, BEC (Business Email Compromise), malicious attachments, or organizations that want to replace or augment a third-party email security gateway.

#### POC Scope Recommendation
- Pilot group of 50–200 mailboxes (representative sample)
- Duration: 2–4 weeks
- Mode: Start in Audit mode before enforcing block policies

#### Prerequisites
- Exchange Online (Microsoft 365 tenant required)
- MDO P2 trial: https://www.microsoft.com/en-us/security/business/email-collaboration-security/microsoft-defender-for-office-365
- Global Admin or Security Admin role

#### Step-by-Step Deployment

**Step 1 — Activate MDO P2 Trial**
```
Microsoft 365 Admin Center → Billing → Purchase services
→ Microsoft Defender for Office 365 P2 (trial)
```

**Step 2 — Configure Preset Security Policies (Recommended)**
```
security.microsoft.com → Email & collaboration → Policies & rules
→ Threat policies → Preset security policies
→ Apply "Standard protection" to pilot group
→ Apply "Strict protection" to high-value targets (executives, finance)
```

This enables with one click:
- Anti-phishing (impersonation protection)
- Safe Links (URL detonation)
- Safe Attachments (file detonation)
- Anti-malware

**Step 3 — Enable Safe Attachments in Audit Mode First**
```
Threat policies → Safe Attachments
→ Create policy → Action: Monitor (audit)
→ Apply to pilot distribution group
→ After 1 week review detections, then switch to Block
```

**Step 4 — Configure Anti-Phishing — Impersonation Protection**
```
Threat policies → Anti-phishing → Edit policy
→ Impersonation protection → Add users to protect (CEO, CFO, Finance team)
→ Add domains to protect (your domain + commonly spoofed domains)
→ Enable mailbox intelligence
```

**Step 5 — Run Attack Simulator**
```
security.microsoft.com → Email & collaboration → Attack simulation training
→ Launch simulation → Credential Harvest → Select payload template
→ Target: pilot user group
→ Duration: 7 days
→ Review click rate and credential submission rate
```

**Step 6 — Review Threat Explorer**
```
security.microsoft.com → Email & collaboration → Explorer
→ Filter: Last 7 days, Malware / Phish
→ Review detected threats, verdict, and action taken
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| Preset policies applied to pilot group | 100% of pilot mailboxes |
| Attack simulation click rate (baseline) | Measured and documented |
| Safe Attachments detonations in audit mode | At least 1 malicious sample detonated |
| Phishing simulation click rate after training | Reduction vs. baseline |
| Zero false positives on legitimate mail | Verified with mail flow review |

#### Official References
- MDO Documentation: https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/
- Preset Security Policies: https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/preset-security-policies
- Attack Simulator: https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/attack-simulation-training-get-started

---

### POC 5: Microsoft Defender for Cloud Apps (MDCA)

**Best for:** Organizations with ungoverned SaaS usage, data leakage concerns, needing CASB capability, or wanting visibility into shadow IT.

#### POC Scope Recommendation
- Integrate with existing proxy/firewall logs OR deploy via Conditional Access App Control
- Duration: 2–3 weeks
- Start with Cloud Discovery (passive) before enforcing session policies

#### Prerequisites
- MDCA license (included in M365 E5 or EMS E5)
- Trial: https://www.microsoft.com/en-us/security/business/cloud-apps-and-saas/microsoft-defender-cloud-apps
- Entra ID P1 (for Conditional Access App Control)
- Firewall/proxy log export capability (for Shadow IT discovery)

#### Step-by-Step Deployment

**Step 1 — Activate Cloud Discovery (Shadow IT)**
```
security.microsoft.com → Cloud apps → Cloud discovery
→ Upload logs from your proxy/firewall (Zscaler, Palo Alto, Cisco, etc.)
→ Or: Enable MDE integration for automatic log collection from endpoints
```

*MDE integration (recommended for MDE customers):*
```
MDCA portal → Settings → Microsoft Defender for Endpoint
→ Enable: Microsoft Defender for Endpoint integration
→ This streams all endpoint web traffic directly to Cloud Discovery
```

**Step 2 — Review Shadow IT Discovery Report**
```
Cloud apps → Cloud discovery → Dashboard
→ Review: Top apps by traffic, risk score, category
→ Identify unsanctioned high-risk apps (e.g., personal cloud storage, AI tools)
→ Tag apps: Sanctioned / Unsanctioned
```

**Step 3 — Connect Sanctioned Apps (API connectors)**
```
Cloud apps → Settings → App connectors
→ Connect: Microsoft 365, Salesforce, Box, Dropbox, GitHub, ServiceNow
→ Each connector enables: Activity logs, DLP scanning, access governance
```

**Step 4 — Configure App Governance**
```
Cloud apps → App governance
→ Review OAuth apps with excessive permissions
→ Identify apps with unused permissions
→ Create alert policy: App requests sensitive permission + low community use score
```

**Step 5 — Deploy Conditional Access App Control (Session Policy)**
```
Entra ID → Conditional Access → Create policy
→ Target: All users / Pilot group
→ Cloud apps: Select 1–2 high-risk apps to pilot
→ Session control: Use Conditional Access App Control
→ MDCA → Policies → Session policy → Block download of sensitive files
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| Shadow IT apps discovered | Inventory of all cloud apps with risk scores |
| Unsanctioned high-risk apps identified | At least 3 identified for review |
| API connectors active | At least Microsoft 365 connected |
| OAuth app overpermission alerts | At least 1 policy active |
| Session policy tested | At least 1 app with block-download policy verified |

#### Official References
- MDCA Documentation: https://learn.microsoft.com/en-us/defender-cloud-apps/
- Cloud Discovery Setup: https://learn.microsoft.com/en-us/defender-cloud-apps/set-up-cloud-discovery
- App Connectors: https://learn.microsoft.com/en-us/defender-cloud-apps/enable-instant-visibility-protection-and-governance-actions-for-your-apps

---

### POC 6: Microsoft Defender for Cloud (MDC)

**Best for:** Organizations running workloads in Azure, AWS, or GCP; teams needing cloud security posture management (CSPM); organizations that want to identify attack paths through cloud environments.

#### POC Scope Recommendation
- Enable on 1–3 Azure subscriptions, or connect 1 AWS account / GCP project
- Duration: 2–3 weeks
- Start with CSPM (free tier) before enabling paid Defender Plans

#### Prerequisites
- Azure subscription (Contributor or Owner role)
- MDC trial: Free CSPM tier is available at no cost; Defender Plans (paid) have 30-day trial
- For AWS/GCP: Cloud connector requires specific IAM roles

#### Step-by-Step Deployment

**Step 1 — Enable Defender for Cloud on Azure Subscription**
```
Azure Portal → Defender for Cloud → Getting started
→ Enable → Select subscription(s) → Upgrade (starts 30-day trial for paid plans)
```

**Step 2 — Review Secure Score (Free CSPM)**
```
Defender for Cloud → Secure Score
→ Review current score (0–100)
→ Click into top recommendations — each shows:
  - Affected resources
  - Remediation steps
  - Estimated score improvement
```

**Step 3 — Enable Defender Plans for POC Resources**
```
Defender for Cloud → Environment settings → Select subscription
→ Enable:
  - Defender for Servers (Plan 2 recommended — includes MDVM)
  - Defender for Containers
  - Defender for SQL
  - Defender for Storage
  - Defender CSPM (enhanced — includes attack path analysis)
```

**Step 4 — Review Attack Path Analysis**
```
Defender for Cloud → Attack path analysis
→ View identified attack paths (e.g., Internet → VM → Storage with PII)
→ Click a path → Review each node and the vulnerability/misconfiguration enabling it
→ Click "Fix" on the highest-impact node to break the path
```

**Step 5 — Enable Agentless Vulnerability Scanning**
```
Environment settings → Defender for Servers Plan 2
→ Agentless scanning: ON
→ Wait 24 hours for initial scan
→ Defender for Cloud → Recommendations → filter: "Machines should have vulnerability findings resolved"
```

**Step 6 — Connect AWS or GCP (Optional)**
```
Defender for Cloud → Environment settings → Add environment → AWS / GCP
→ Follow connector wizard:
  - AWS: Deploy CloudFormation stack (auto-creates IAM role with read-only permissions)
  - GCP: Deploy Terraform template (creates service account with required roles)
→ Multi-cloud findings appear alongside Azure findings in unified dashboard
```

**Step 7 — Configure Defender for DevOps**
```
Defender for Cloud → DevOps security → Add environment
→ Connect: GitHub, Azure DevOps, GitLab
→ Review IaC findings, secret scanning results, code security posture
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| Secure Score baseline established | Score visible with top 5 recommendations |
| Agentless scan completed | CVE findings on all POC VMs |
| Attack path(s) identified | At least 1 critical path documented |
| Defender Plans enabled and alerting | At least 1 simulated alert triggered |
| Multi-cloud connected (if applicable) | AWS or GCP findings visible |

#### Official References
- MDC Documentation: https://learn.microsoft.com/en-us/azure/defender-for-cloud/
- Enable Defender for Cloud: https://learn.microsoft.com/en-us/azure/defender-for-cloud/get-started
- Attack Path Analysis: https://learn.microsoft.com/en-us/azure/defender-for-cloud/concept-attack-path
- Multi-cloud Connectors: https://learn.microsoft.com/en-us/azure/defender-for-cloud/plan-multicloud-security-get-started

---

### POC 7: Microsoft Sentinel

**Best for:** Organizations building or modernizing a SOC, replacing a legacy SIEM, needing cloud-native threat hunting, or wanting to aggregate alerts from all Defender products and third-party tools into a single platform.

#### POC Scope Recommendation
- Connect Microsoft 365 Defender and at least one third-party source
- Duration: 3–4 weeks (analytics rules need time to generate signal)
- Estimate ingestion volume before enabling — use the cost estimator

#### Prerequisites
- Azure subscription
- Log Analytics Workspace (Sentinel is deployed on top of it)
- Microsoft 365 E5 or Defender XDR licenses (for native connectors)
- Sentinel trial: First 31 days of data ingestion are free up to 10 GB/day

#### Step-by-Step Deployment

**Step 1 — Deploy Sentinel Workspace**
```
Azure Portal → Microsoft Sentinel → Create
→ Create new Log Analytics workspace (or use existing)
→ Region: Select closest to your data residency requirement
→ Pricing tier: Pay-as-you-go (or Commitment tier for >100 GB/day)
```

**Step 2 — Connect Microsoft 365 Defender (Native Connector)**
```
Sentinel → Data connectors → Microsoft Defender XDR
→ Connect → Enable all components:
  ✓ Microsoft Defender for Endpoint alerts and events
  ✓ Microsoft Defender for Office 365 alerts
  ✓ Microsoft Defender for Identity alerts
  ✓ Microsoft Defender for Cloud Apps alerts
  ✓ Microsoft Defender Vulnerability Management findings
```

**Step 3 — Connect Defender for Cloud**
```
Data connectors → Microsoft Defender for Cloud
→ Connect subscription(s)
→ Enable: Alerts stream to Sentinel
```

**Step 4 — Connect Third-Party Sources**
```
Data connectors → Search for your tools:
→ Tenable.io → Connect (requires Tenable API key)
→ CrowdStrike Falcon → Connect
→ Cisco ASA / Palo Alto / Fortinet → Connect via Syslog/CEF
→ Windows Security Events via AMA → Connect domain controllers
```

**Step 5 — Enable Analytics Rules**
```
Sentinel → Analytics → Rule templates
→ Filter: Severity = High, Status = Enabled
→ Enable recommended rules for connected data sources
→ Start with Microsoft Security rules (pre-built, no tuning required)
```

**Step 6 — Enable UEBA (User and Entity Behavior Analytics)**
```
Sentinel → Settings → Entity behavior
→ Enable UEBA
→ Sync from: Microsoft Entra ID, Active Directory
→ Wait 24–48 hours for baselines to build
```

**Step 7 — Create Workbooks for Vulnerability Reporting**
```
Sentinel → Workbooks → Add workbook
→ Templates: search "vulnerability"
→ Enable: Microsoft Defender Vulnerability Management workbook
→ Customize to show: MTTR, SLA compliance, KEV coverage, scan coverage
```

**Step 8 — Create a SOAR Playbook**
```
Sentinel → Automation → Create playbook
→ Trigger: When a High/Critical incident is created
→ Actions:
  1. Post message to Teams security channel
  2. Create ServiceNow ticket
  3. Add comment to incident with asset owner from CMDB
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| Data connectors active and ingesting | At least 3 sources connected |
| Incidents generated from analytics rules | At least 5 incidents in first week |
| UEBA anomalies detected | At least 1 anomaly surfaced |
| Vulnerability workbook operational | MTTR and exposure data visible |
| SOAR playbook tested | End-to-end: alert → Teams notification → ticket |
| Mean time to close test incident | Measured as baseline |

#### Official References
- Sentinel Documentation: https://learn.microsoft.com/en-us/azure/sentinel/
- Deploy Sentinel: https://learn.microsoft.com/en-us/azure/sentinel/quickstart-onboard
- Data Connectors: https://learn.microsoft.com/en-us/azure/sentinel/connect-data-sources
- Analytics Rules: https://learn.microsoft.com/en-us/azure/sentinel/detect-threats-built-in
- SOAR Playbooks: https://learn.microsoft.com/en-us/azure/sentinel/automate-responses-with-playbooks

---

### POC 8: Microsoft Security Copilot

**Best for:** SOC teams overwhelmed with alert volume, analysts spending too long on investigation, organizations wanting AI-assisted vulnerability triage and remediation, security managers needing executive-ready reports quickly.

#### POC Scope Recommendation
- No infrastructure deployment — Copilot works on top of existing Defender products
- Start with 2–5 security analysts for 2–3 weeks
- Requires at least one Defender product active (MDE, MDI, Sentinel, etc.)

#### Prerequisites
- Security Copilot license (SCU-based): https://learn.microsoft.com/en-us/security-copilot/get-started-security-copilot
- At least one Defender product connected as a plugin
- Security Admin or Global Admin role to activate

#### Step-by-Step Deployment

**Step 1 — Provision Security Copilot**
```
https://securitycopilot.microsoft.com → Get started
→ Select Azure subscription for billing
→ Choose region
→ Allocate SCUs (Security Compute Units) — start with 1–3 SCUs for POC
```

**Step 2 — Connect Plugins**
```
Copilot portal → Sources (plugin icon)
→ Enable:
  ✓ Microsoft Defender XDR
  ✓ Microsoft Sentinel
  ✓ Microsoft Defender for Cloud
  ✓ Microsoft Intune
  ✓ Microsoft Entra
  ✓ MSRC (Security Update Guide)
  ✓ NVD / EPSS (public threat intelligence)
```

**Step 3 — Run Vulnerability POC Prompts**

Test these prompts to validate CVE/vulnerability capability:

```
"Summarize CVE-2024-21413 and tell me which devices in my environment are affected"

"What are the top 5 critical vulnerabilities in my environment right now based on MSRC severity and EPSS?"

"Generate a PowerShell script to check if KB5034441 is installed on all domain-joined Windows 11 devices"

"Create an executive summary of this month's Patch Tuesday advisories highlighting the CVEs most relevant to our environment"

"Is CVE-2024-38080 being actively exploited? What's the MSRC Exploitability Index and our exposure?"
```

**Step 4 — Run Incident Investigation Prompts**

```
"Summarize incident INC-00123 and list all affected entities"

"What MITRE ATT&CK techniques were used in this incident?"

"Generate a timeline of events for this incident"

"What is the recommended containment action for this incident?"
```

**Step 5 — Measure Analyst Time Savings**
```
Before POC: Record time to:
  - Triage a High severity alert: ____ minutes
  - Write an incident summary: ____ minutes
  - Identify affected assets for a CVE: ____ minutes

After POC: Remeasure same tasks with Copilot assistance
Target: 30–50% time reduction
```

#### POC Success Criteria
| Criterion | Target |
|---|---|
| All Defender plugins connected | 100% of available plugins enabled |
| CVE summarization accuracy | Validated against NVD for 5 test CVEs |
| Affected device query response | Returns correct device list |
| Remediation script generated | Reviewed and approved by engineer |
| Analyst time savings | > 30% reduction in triage time |
| Executive report generated | Draft produced in < 2 minutes |

#### Official References
- Security Copilot: https://learn.microsoft.com/en-us/security-copilot/
- Get Started: https://learn.microsoft.com/en-us/security-copilot/get-started-security-copilot
- Promptbook Library: https://learn.microsoft.com/en-us/security-copilot/using-promptbooks
- Plugin Configuration: https://learn.microsoft.com/en-us/security-copilot/manage-plugins

---

## Scenario-Based POC Recommendations

### How to Use This Section

When a user describes their situation, match it to the scenarios below and provide the recommended POC playbook(s).

---

### Scenario A: "We had a ransomware incident and our endpoint protection failed"

**Recommended Products:**
1. **Defender for Endpoint P2** (primary — EDR + automatic investigation)
2. **Defender Vulnerability Management** (secondary — identify exploited CVEs)
3. **Defender XDR** (unified response across endpoints and identity)

**POC Focus:**
- Deploy MDE on all endpoints (POC 1)
- Run attack simulation to validate EDR detection (Step 4 of POC 1)
- Enable Attack Surface Reduction rules to prevent re-infection vectors
- Review MDVM for CVEs that the ransomware may have exploited

---

### Scenario B: "We don't know what patches are missing across our environment"

**Recommended Products:**
1. **Defender Vulnerability Management** (primary)
2. **Defender for Endpoint P2** (required — MDVM runs on MDE)
3. **Microsoft Intune + WSUS** (remediation deployment)

**POC Focus:**
- Onboard endpoints to MDE (POC 1)
- Enable MDVM and review Exposure Score (POC 2)
- Export top 20 recommendations and create remediation activities
- Validate MSRC KB linkage for Windows CVEs

---

### Scenario C: "Our Active Directory may be compromised — we see unusual login patterns"

**Recommended Products:**
1. **Defender for Identity** (primary — AD threat detection)
2. **Microsoft Sentinel** (secondary — correlation with other signals)
3. **Defender XDR** (unified incident view)

**POC Focus:**
- Install MDI sensors on all DCs immediately (POC 3)
- Run lateral movement path analysis
- Connect Sentinel for correlated incident view (POC 7)
- Review Entra ID sign-in risk reports alongside MDI alerts

---

### Scenario D: "Our users keep clicking phishing emails and we need better email security"

**Recommended Products:**
1. **Defender for Office 365 P2** (primary)
2. **Attack Simulation Training** (included in MDO P2)
3. **Defender XDR** (unified investigation)

**POC Focus:**
- Enable MDO preset security policies (POC 4)
- Run baseline phishing simulation to measure click rate (Step 5 of POC 4)
- Deliver security awareness training to high-click users
- Measure click rate reduction after 30 days

---

### Scenario E: "We have no visibility into what cloud apps our employees are using"

**Recommended Products:**
1. **Defender for Cloud Apps** (primary — CASB + Shadow IT)
2. **Defender for Endpoint** (for MDE-based log collection)
3. **Entra ID Conditional Access** (for session control policies)

**POC Focus:**
- Enable Cloud Discovery via MDE integration (POC 5, Step 1)
- Review shadow IT report (Step 2)
- Connect Microsoft 365 and Salesforce API connectors (Step 3)
- Identify OAuth apps with excessive permissions (Step 4)

---

### Scenario F: "Our cloud infrastructure in Azure/AWS is misconfigured and we had a data breach"

**Recommended Products:**
1. **Defender for Cloud** (primary — CSPM + attack path analysis)
2. **Microsoft Sentinel** (secondary — SIEM correlation)
3. **Defender for DevOps** (shift-left — prevent misconfigurations from reaching production)

**POC Focus:**
- Enable MDC on all subscriptions (POC 6)
- Run attack path analysis immediately (Step 4 of POC 6)
- Enable agentless vulnerability scanning (Step 5)
- Connect AWS/GCP if multi-cloud (Step 6)
- Integrate with Sentinel for unified alerting (POC 7)

---

### Scenario G: "Our SOC analysts are overwhelmed — too many alerts, not enough time"

**Recommended Products:**
1. **Microsoft Sentinel** (primary — SIEM/SOAR, automation)
2. **Microsoft Security Copilot** (secondary — AI-assisted triage)
3. **Defender XDR** (auto-investigation and auto-remediation)

**POC Focus:**
- Deploy Sentinel and connect all Defender products (POC 7)
- Enable SOAR playbooks for automatic ticket creation and notification (Step 8)
- Provision Security Copilot for analyst-assisted investigation (POC 8)
- Measure analyst time-to-triage before and after (Step 5 of POC 8)

---

### Scenario H: "We need to prove security ROI to our board / get budget approved"

**Recommended Products:**
1. **Microsoft Security Copilot** (generates executive reports)
2. **Defender Vulnerability Management** (exposure score = quantifiable risk reduction)
3. **Defender for Cloud** (Secure Score = measurable posture improvement)

**POC Focus:**
- Establish baseline: Exposure Score + Secure Score + open Critical/High CVE count
- Run 30-day POC with remediation activities
- Use Security Copilot to generate executive summary report
- Present: Before/after Exposure Score, CVEs closed, MTTR improvement

---

### Scenario I: "We are building a Zero Trust architecture from scratch"

**Recommended Products (full stack):**
1. **MDE** — Device compliance and health signals
2. **MDI** — Identity risk signals
3. **Entra ID P2** — Conditional Access + Identity Protection
4. **MDCA** — App access control + session policies
5. **MDC** — Cloud workload protection
6. **Sentinel** — Unified monitoring

**POC Focus (phased):**
- Phase 1 (Identity): MDI + Entra Conditional Access
- Phase 2 (Endpoints): MDE + device compliance policies
- Phase 3 (Apps): MDCA + Conditional Access App Control
- Phase 4 (Cloud): MDC + Sentinel for unified visibility
- Each phase validated before proceeding to next

---

## POC Engagement Template

When a user presents a scenario, respond using this structure:

```
## Microsoft Defender POC Recommendation

### Scenario Summary
[Restate the user's scenario in 2–3 sentences]

### Recommended Product(s)
| Priority | Product | Reason |
|---|---|---|
| Primary | [Product] | [Why this product addresses the scenario] |
| Secondary | [Product] | [Complementary capability] |

### POC Scope
- Devices / users / resources to cover: [specific guidance]
- Duration: [X weeks]
- Mode: [Audit first / Direct enforcement]
- License required: [Trial URL]

### Deployment Steps (Summary)
1. [Step 1]
2. [Step 2]
3. [Step 3]
...

### Success Criteria
| Criterion | Target |
|---|---|
| [Metric] | [Target value] |

### Full POC Playbook
→ See POC [N]: [Product Name] in microsoft-defender-poc.md

### Official References
- [Product docs URL]
- [Trial URL]
```

---

## Official References

| Resource | URL |
|---|---|
| Microsoft Defender for Endpoint | https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/ |
| Microsoft Defender Vulnerability Management | https://learn.microsoft.com/en-us/defender-vulnerability-management/ |
| Microsoft Defender for Identity | https://learn.microsoft.com/en-us/defender-for-identity/ |
| Microsoft Defender for Office 365 | https://learn.microsoft.com/en-us/microsoft-365/security/office-365-security/ |
| Microsoft Defender for Cloud Apps | https://learn.microsoft.com/en-us/defender-cloud-apps/ |
| Microsoft Defender for Cloud | https://learn.microsoft.com/en-us/azure/defender-for-cloud/ |
| Microsoft Sentinel | https://learn.microsoft.com/en-us/azure/sentinel/ |
| Microsoft Security Copilot | https://learn.microsoft.com/en-us/security-copilot/ |
| Microsoft Defender XDR | https://learn.microsoft.com/en-us/microsoft-365/security/defender/ |
| Microsoft Defender for IoT | https://learn.microsoft.com/en-us/azure/defender-for-iot/ |
| Microsoft Defender for DevOps | https://learn.microsoft.com/en-us/azure/defender-for-cloud/defender-for-devops-introduction |
| Microsoft 365 Trials | https://www.microsoft.com/en-us/microsoft-365/enterprise/compare-office-365-plans |
| Microsoft Defender XDR Trial | https://www.microsoft.com/en-us/security/business/microsoft-defender |
| MSRC Security Update Guide | https://msrc.microsoft.com/update-guide/ |
| Zero Trust Guidance Center | https://learn.microsoft.com/en-us/security/zero-trust/ |
