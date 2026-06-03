# Defender Alert Analysis — Microsoft Security Graph API, MITRE ATT&CK Mapping, and Severity Assessment

## Overview

This reference enables Arcus to interpret and explain Microsoft Defender threat alerts by:

1. **Ingesting alert data** from the Microsoft Security Graph API
2. **Mapping every alert** to MITRE ATT&CK Tactics, Techniques, and Sub-techniques
3. **Assessing severity** using a composite model combining Defender severity, MITRE impact, asset criticality, and environmental context
4. **Explaining the alert** in plain language for both technical and non-technical audiences
5. **Recommending response actions** aligned to the VMLC ACT and REASSESS phases

---

## Part 1 — Microsoft Security Graph API

### Overview

The Microsoft Security Graph API (`https://graph.microsoft.com/v1.0/security/`) is the unified programmatic interface to all Microsoft Defender security signals. It exposes alerts, incidents, threat intelligence indicators, secure scores, and vulnerability findings from:

- Microsoft Defender for Endpoint (MDE)
- Microsoft Defender for Identity (MDI)
- Microsoft Defender for Office 365 (MDO)
- Microsoft Defender for Cloud Apps (MDCA)
- Microsoft Defender for Cloud (MDC)
- Microsoft Sentinel
- Microsoft Entra ID Protection

**API Base URL:** `https://graph.microsoft.com/v1.0/security/`
**Beta (preview features):** `https://graph.microsoft.com/beta/security/`
**Official Docs:** https://learn.microsoft.com/en-us/graph/api/resources/security-api-overview

---

### Authentication

The Security Graph API uses **OAuth 2.0 with Microsoft Entra ID**. Two authentication flows are supported:

#### Option A — Application (Daemon/Service) — Recommended for automation

```python
# Python — MSAL library
import msal, requests

TENANT_ID    = "your-tenant-id"
CLIENT_ID    = "your-app-registration-client-id"
CLIENT_SECRET= "your-client-secret"

app = msal.ConfidentialClientApplication(
    CLIENT_ID,
    authority=f"https://login.microsoftonline.com/{TENANT_ID}",
    client_credential=CLIENT_SECRET
)

token = app.acquire_token_for_client(
    scopes=["https://graph.microsoft.com/.default"]
)
access_token = token["access_token"]
```

```powershell
# PowerShell — client credentials flow
$body = @{
    grant_type    = "client_credentials"
    client_id     = $clientId
    client_secret = $clientSecret
    scope         = "https://graph.microsoft.com/.default"
}
$token = Invoke-RestMethod `
    -Uri "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token" `
    -Method POST -Body $body
$accessToken = $token.access_token
```

#### Option B — Delegated (Interactive / on behalf of user)

Use when running as a signed-in analyst in a tool or script:
```
Scopes required: SecurityAlert.Read.All, SecurityIncident.Read.All, ThreatIndicators.Read.All
```

#### Required API Permissions

| Permission | Type | Purpose |
|---|---|---|
| `SecurityAlert.Read.All` | Application | Read all security alerts |
| `SecurityAlert.ReadWrite.All` | Application | Read and update alerts |
| `SecurityIncident.Read.All` | Application | Read incidents (grouped alerts) |
| `SecurityIncident.ReadWrite.All` | Application | Update incident status |
| `ThreatIndicators.Read.All` | Application | Read threat intelligence indicators |
| `SecurityEvents.Read.All` | Application | Read raw security events |
| `IdentityRiskEvent.Read.All` | Application | Read Entra ID risky sign-ins |

**Grant permissions:** Azure Portal → App Registrations → Your App → API Permissions → Add permission → Microsoft Graph → Application permissions → Grant admin consent

---

### Core API Endpoints

#### Alerts v2 (Recommended — unified alert model)

```http
GET https://graph.microsoft.com/v1.0/security/alerts_v2
Authorization: Bearer {access_token}
```

**Filter examples:**

```http
# High and Critical alerts only
GET https://graph.microsoft.com/v1.0/security/alerts_v2?$filter=severity eq 'high' or severity eq 'critical'

# Alerts from last 24 hours
GET https://graph.microsoft.com/v1.0/security/alerts_v2?$filter=createdDateTime ge 2024-02-13T00:00:00Z

# Alerts for a specific device
GET https://graph.microsoft.com/v1.0/security/alerts_v2?$filter=evidence/any(e:e/microsoft.graph.security.deviceEvidence/deviceDnsName eq 'srv-dc01.contoso.com')

# Unresolved alerts sorted by severity
GET https://graph.microsoft.com/v1.0/security/alerts_v2?$filter=status ne 'resolved'&$orderby=severity desc&$top=50

# Single alert by ID
GET https://graph.microsoft.com/v1.0/security/alerts_v2/{alertId}
```

**Alert v2 response schema:**

```json
{
  "id": "da637551227677560813_-961444813",
  "providerAlertId": "da637551227677560813_-961444813",
  "incidentId": "28282",
  "status": "new",
  "severity": "high",
  "classification": null,
  "determination": null,
  "serviceSource": "microsoftDefenderForEndpoint",
  "detectionSource": "antivirus",
  "tenantId": "b3c1b5fc-828c-45fa-a1e1-10d74f6d6e9c",
  "title": "Ransomware behavior blocked",
  "description": "Ransomware-like behavior was detected and blocked on the device.",
  "recommendedActions": "Investigate the alert and apply recommended remediation actions.",
  "category": "Ransomware",
  "assignedTo": null,
  "alertWebUrl": "https://security.microsoft.com/alerts/da637551227677560813_-961444813",
  "incidentWebUrl": "https://security.microsoft.com/incidents/28282",
  "actorDisplayName": null,
  "threatDisplayName": "Ransom:Win32/Phobos",
  "threatFamilyName": "Phobos",
  "mitreTechniques": ["T1486", "T1490"],
  "createdDateTime": "2024-02-13T08:22:47.527Z",
  "lastUpdateDateTime": "2024-02-13T08:43:15.433Z",
  "resolvedDateTime": null,
  "firstActivityDateTime": "2024-02-13T08:14:55.000Z",
  "lastActivityDateTime": "2024-02-13T08:22:40.000Z",
  "evidence": [
    {
      "@odata.type": "#microsoft.graph.security.deviceEvidence",
      "deviceDnsName": "srv-prod-01.contoso.com",
      "osPlatform": "Windows11",
      "rbacGroupName": "Production Servers",
      "onboardingStatus": "onboarded",
      "defenderAvStatus": "updated",
      "healthStatus": "active"
    },
    {
      "@odata.type": "#microsoft.graph.security.fileEvidence",
      "fileName": "invoice.exe",
      "filePath": "C:\\Users\\jdoe\\Downloads\\invoice.exe",
      "fileSize": 204800,
      "sha256": "a3b2c1d4e5f6...",
      "issuer": null,
      "signer": null,
      "globalPrevalence": 12,
      "globalFirstSeen": "2024-02-12T19:00:00Z"
    }
  ]
}
```

#### Incidents (Correlated Alert Groups)

```http
GET https://graph.microsoft.com/v1.0/security/incidents
Authorization: Bearer {access_token}

# Incident with all alerts expanded
GET https://graph.microsoft.com/v1.0/security/incidents/{incidentId}?$expand=alerts

# High severity active incidents
GET https://graph.microsoft.com/v1.0/security/incidents?$filter=severity eq 'high' and status ne 'resolved'&$orderby=createdDateTime desc
```

**Update alert status (close/resolve):**

```http
PATCH https://graph.microsoft.com/v1.0/security/alerts_v2/{alertId}
Content-Type: application/json

{
  "status": "resolved",
  "classification": "truePositive",
  "determination": "malware",
  "assignedTo": "analyst@contoso.com",
  "comment": "Confirmed ransomware. Device isolated and reimaged. CVE-2024-XXXX was exploited."
}
```

#### Threat Intelligence Indicators

```http
# Get all threat intelligence indicators
GET https://graph.microsoft.com/v1.0/security/threatIntelligence/indicators

# Get indicator by observable (IP, domain, hash)
GET https://graph.microsoft.com/v1.0/security/threatIntelligence/indicators?$filter=artifact/microsoft.graph.security.hostname/hostName eq 'malicious-domain.com'
```

#### Secure Score

```http
GET https://graph.microsoft.com/v1.0/security/secureScores?$top=1
GET https://graph.microsoft.com/v1.0/security/secureScoreControlProfiles
```

---

### Full Python Alert Ingestion Example

```python
import msal
import requests
import json
from datetime import datetime, timedelta, timezone

TENANT_ID     = "your-tenant-id"
CLIENT_ID     = "your-client-id"
CLIENT_SECRET = "your-client-secret"
GRAPH_BASE    = "https://graph.microsoft.com/v1.0"

def get_token():
    app = msal.ConfidentialClientApplication(
        CLIENT_ID,
        authority=f"https://login.microsoftonline.com/{TENANT_ID}",
        client_credential=CLIENT_SECRET
    )
    result = app.acquire_token_for_client(
        scopes=["https://graph.microsoft.com/.default"]
    )
    if "access_token" not in result:
        raise Exception(f"Auth failed: {result.get('error_description')}")
    return result["access_token"]

def get_recent_alerts(hours=24, severity_filter=None):
    token = get_token()
    headers = {"Authorization": f"Bearer {token}", "Content-Type": "application/json"}

    since = (datetime.now(timezone.utc) - timedelta(hours=hours)).isoformat()
    filt = f"createdDateTime ge {since}"
    if severity_filter:
        sev_clause = " or ".join([f"severity eq '{s}'" for s in severity_filter])
        filt += f" and ({sev_clause})"

    url = f"{GRAPH_BASE}/security/alerts_v2?$filter={filt}&$orderby=severity desc&$top=100"
    alerts = []
    while url:
        resp = requests.get(url, headers=headers)
        resp.raise_for_status()
        data = resp.json()
        alerts.extend(data.get("value", []))
        url = data.get("@odata.nextLink")  # handle pagination
    return alerts

def enrich_with_mitre(alerts):
    """Add MITRE ATT&CK details to each alert."""
    enriched = []
    for alert in alerts:
        techniques = alert.get("mitreTechniques", [])
        alert["mitreEnriched"] = [get_mitre_details(t) for t in techniques]
        enriched.append(alert)
    return enriched

def get_mitre_details(technique_id):
    """Look up MITRE technique details via MITRE ATT&CK TAXII or local mapping."""
    return MITRE_TECHNIQUE_MAP.get(technique_id, {
        "id": technique_id,
        "name": "Unknown",
        "tactic": "Unknown",
        "url": f"https://attack.mitre.org/techniques/{technique_id.replace('.', '/')}/"
    })

if __name__ == "__main__":
    alerts = get_recent_alerts(hours=24, severity_filter=["high", "critical"])
    alerts = enrich_with_mitre(alerts)
    print(json.dumps(alerts, indent=2, default=str))
```

---

### PowerShell Alert Ingestion Example

```powershell
param(
    [string]$TenantId,
    [string]$ClientId,
    [string]$ClientSecret,
    [int]$HoursBack = 24,
    [string[]]$Severities = @("high","critical")
)

# Authenticate
$tokenBody = @{
    grant_type    = "client_credentials"
    client_id     = $ClientId
    client_secret = $ClientSecret
    scope         = "https://graph.microsoft.com/.default"
}
$token = (Invoke-RestMethod `
    -Uri "https://login.microsoftonline.com/$TenantId/oauth2/v2.0/token" `
    -Method POST -Body $tokenBody).access_token

$headers = @{ Authorization = "Bearer $token" }
$since   = (Get-Date).ToUniversalTime().AddHours(-$HoursBack).ToString("o")

# Build filter
$sevFilter = ($Severities | ForEach-Object { "severity eq '$_'" }) -join " or "
$filter    = [uri]::EscapeDataString("createdDateTime ge $since and ($sevFilter)")
$uri       = "https://graph.microsoft.com/v1.0/security/alerts_v2?`$filter=$filter&`$orderby=severity desc&`$top=100"

# Paginate through all results
$alerts = @()
do {
    $response = Invoke-RestMethod -Uri $uri -Headers $headers
    $alerts  += $response.value
    $uri      = $response.'@odata.nextLink'
} while ($uri)

Write-Host "Retrieved $($alerts.Count) alert(s)"
$alerts | Select-Object id, title, severity, status, category, mitreTechniques, createdDateTime |
    Format-Table -AutoSize
```

---

## Part 2 — MITRE ATT&CK Framework Mapping

### Overview

MITRE ATT&CK is a globally accessible knowledge base of adversary tactics, techniques, and sub-techniques based on real-world observations. Microsoft Defender alerts include `mitreTechniques` arrays that directly reference ATT&CK technique IDs.

**ATT&CK Matrix versions:**
- **Enterprise** — Windows, macOS, Linux, cloud, containers, network
- **Mobile** — iOS, Android
- **ICS** — Industrial Control Systems

**Official Reference:** https://attack.mitre.org/
**TAXII API:** https://attack.mitre.org/resources/attack-data-and-tools/
**STIX/TAXII Server:** `https://attack-taxii.mitre.org/`

---

### ATT&CK Structure

```
Tactic (Why — adversary goal)
  └── Technique (How — general method)         T####
        └── Sub-technique (Specific variant)   T####.###
```

**14 Enterprise Tactics:**

| ID | Tactic | Description |
|---|---|---|
| TA0043 | Reconnaissance | Gathering info before the attack |
| TA0042 | Resource Development | Acquiring infrastructure, tools, accounts |
| TA0001 | Initial Access | First foothold into the environment |
| TA0002 | Execution | Running malicious code |
| TA0003 | Persistence | Maintaining foothold across restarts |
| TA0004 | Privilege Escalation | Gaining higher-level permissions |
| TA0005 | Defense Evasion | Avoiding detection |
| TA0006 | Credential Access | Stealing account credentials |
| TA0007 | Discovery | Mapping the environment |
| TA0008 | Lateral Movement | Moving through the network |
| TA0009 | Collection | Gathering data of interest |
| TA0010 | Exfiltration | Stealing data out of the environment |
| TA0011 | Command and Control | Communicating with compromised systems |
| TA0040 | Impact | Manipulating, interrupting, or destroying systems |

---

### High-Frequency Defender Alert → MITRE Technique Mapping

This table maps the most common Defender alert categories to their MITRE techniques:

| Defender Alert Category | MITRE Technique ID | Technique Name | Tactic |
|---|---|---|---|
| Ransomware behavior | T1486 | Data Encrypted for Impact | Impact (TA0040) |
| Ransomware — shadow copy deletion | T1490 | Inhibit System Recovery | Impact (TA0040) |
| Credential dumping — LSASS | T1003.001 | OS Credential Dumping: LSASS Memory | Credential Access (TA0006) |
| Credential dumping — SAM | T1003.002 | OS Credential Dumping: SAM | Credential Access (TA0006) |
| Pass-the-Hash | T1550.002 | Use Alternate Auth Material: Pass the Hash | Lateral Movement (TA0008) |
| Pass-the-Ticket / Kerberoasting | T1558.003 | Steal/Forge Kerberos Tickets: Kerberoasting | Credential Access (TA0006) |
| Golden Ticket | T1558.001 | Steal/Forge Kerberos Tickets: Golden Ticket | Credential Access (TA0006) |
| PowerShell execution | T1059.001 | Command and Scripting Interpreter: PowerShell | Execution (TA0002) |
| WMI execution | T1047 | Windows Management Instrumentation | Execution (TA0002) |
| Scheduled task persistence | T1053.005 | Scheduled Task/Job: Scheduled Task | Persistence (TA0003) |
| Registry run key persistence | T1547.001 | Boot/Logon Autostart: Registry Run Keys | Persistence (TA0003) |
| Process injection | T1055 | Process Injection | Defense Evasion (TA0005) / Privilege Escalation (TA0004) |
| LOLBAS / living off the land | T1218 | System Binary Proxy Execution | Defense Evasion (TA0005) |
| Obfuscated scripts | T1027 | Obfuscated Files or Information | Defense Evasion (TA0005) |
| Phishing — malicious attachment | T1566.001 | Phishing: Spearphishing Attachment | Initial Access (TA0001) |
| Phishing — malicious link | T1566.002 | Phishing: Spearphishing Link | Initial Access (TA0001) |
| Drive-by compromise | T1189 | Drive-by Compromise | Initial Access (TA0001) |
| Exploit public-facing application | T1190 | Exploit Public-Facing Application | Initial Access (TA0001) |
| Lateral movement via SMB | T1021.002 | Remote Services: SMB/Windows Admin Shares | Lateral Movement (TA0008) |
| Lateral movement via RDP | T1021.001 | Remote Services: Remote Desktop Protocol | Lateral Movement (TA0008) |
| Network scanning / discovery | T1046 | Network Service Discovery | Discovery (TA0007) |
| Account enumeration | T1087 | Account Discovery | Discovery (TA0007) |
| Data exfiltration | T1041 | Exfiltration Over C2 Channel | Exfiltration (TA0010) |
| DNS tunneling (C2) | T1071.004 | Application Layer Protocol: DNS | Command and Control (TA0011) |
| HTTPS C2 | T1071.001 | Application Layer Protocol: Web Protocols | Command and Control (TA0011) |
| Token impersonation | T1134 | Access Token Manipulation | Privilege Escalation (TA0004) |
| UAC bypass | T1548.002 | Abuse Elevation Control Mechanism: UAC Bypass | Defense Evasion (TA0005) |
| DLL side-loading | T1574.002 | Hijack Execution Flow: DLL Side-Loading | Persistence / Defense Evasion |
| Suspicious service creation | T1543.003 | Create or Modify System Process: Windows Service | Persistence (TA0003) |
| Shadow IT / cloud app anomaly | T1537 | Transfer Data to Cloud Account | Exfiltration (TA0010) |
| Brute force login | T1110 | Brute Force | Credential Access (TA0006) |
| Password spray | T1110.003 | Brute Force: Password Spraying | Credential Access (TA0006) |
| Impossible travel (Entra ID) | T1078 | Valid Accounts | Defense Evasion / Initial Access |
| Malicious OAuth app | T1550.001 | Use Alternate Auth Material: App Access Token | Defense Evasion (TA0005) |

---

### MITRE ATT&CK TAXII API — Programmatic Lookup

```python
from taxii2client.v20 import Server
import json

# Connect to MITRE ATT&CK TAXII server
server = Server("https://attack-taxii.mitre.org/")
api_root = server.api_roots[0]

# List available collections (Enterprise, Mobile, ICS)
for collection in api_root.collections:
    print(collection.id, collection.title)

# Enterprise ATT&CK collection ID
ENTERPRISE_COLLECTION_ID = "x-mitre-collection--1f5f1533-f617-4ca8-9ab4-6a02367fa019"
```

**Simpler option — ATT&CK Python library:**

```python
# pip install mitreattack-python
from mitreattack.stix20 import MitreAttackData

# Load Enterprise ATT&CK
mitre = MitreAttackData("enterprise-attack.json")  # download from attack.mitre.org

def lookup_technique(technique_id):
    technique = mitre.get_object_by_attack_id(technique_id, "attack-pattern")
    if not technique:
        return None
    tactics = [phase["phase_name"] for phase in technique.get("kill_chain_phases", [])]
    return {
        "id": technique_id,
        "name": technique["name"],
        "description": technique.get("description", "")[:300],
        "tactics": tactics,
        "url": f"https://attack.mitre.org/techniques/{technique_id.replace('.', '/')}/"
    }

# Example
print(lookup_technique("T1486"))
# → {"id": "T1486", "name": "Data Encrypted for Impact", "tactics": ["impact"], ...}
```

---

### Alert MITRE Enrichment Response Format

When Arcus receives an alert (raw or via API), it must produce this structured MITRE enrichment output:

```
## MITRE ATT&CK Analysis

Alert: [Alert Title]
Source: [Defender product — MDE / MDI / MDO / MDCA / MDC]

### Techniques Detected

| Technique ID | Technique Name | Tactic | Sub-technique of |
|---|---|---|---|
| T1486 | Data Encrypted for Impact | Impact (TA0040) | — |
| T1490 | Inhibit System Recovery | Impact (TA0040) | — |

### Kill Chain Position

[Reconstruct the likely attack sequence using the detected techniques]

Reconnaissance → Initial Access (T1566.001 Phishing)
  → Execution (T1059.001 PowerShell)
    → Privilege Escalation (T1055 Process Injection)
      → Credential Access (T1003.001 LSASS Dump)
        → Lateral Movement (T1021.002 SMB)
          → Impact (T1486 Ransomware Encryption)

### Adversary Objective
[What the attacker was attempting to achieve at this stage]

### Known Threat Actors Using These Techniques
[Reference ATT&CK Groups if applicable — e.g., G0102 Wizard Spider uses T1486]

### ATT&CK Navigator Layer
[Provide the technique IDs in a format compatible with ATT&CK Navigator]
https://mitre-attack.github.io/attack-navigator/
```

---

## Part 3 — Severity Assessment Model

### Overview

Microsoft Defender assigns a raw severity (`informational`, `low`, `medium`, `high`, `critical`) based on the detection engine's confidence and the known impact of the detected behavior. Arcus applies a **composite severity assessment** that enriches this with MITRE impact weight, asset criticality, environmental context, and active exploitation signals.

---

### Defender Native Severity Definitions

| Defender Severity | Description | Typical Alerts |
|---|---|---|
| **Critical** | Immediate threat to critical assets — active compromise, active data exfiltration, ransomware | Active ransomware, Golden Ticket, LSASS dump on DC |
| **High** | Significant threat — sophisticated techniques, lateral movement in progress | Pass-the-Hash, credential dumping, C2 communication |
| **Medium** | Potentially malicious — requires investigation to confirm | Suspicious PowerShell, unsigned driver loading, anomalous login |
| **Low** | Minor anomaly — likely benign but warrants awareness | Unusual network connection, low-prevalence file |
| **Informational** | Audit/telemetry — no immediate threat | Audit log entries, policy changes, configuration drifts |

---

### Composite Severity Score

Arcus computes a **Composite Severity Score (CSS)** from 0–100 for every alert:

```
CSS = (Defender Severity Score × 0.30)
    + (MITRE Tactic Impact Weight × 0.25)
    + (Asset Criticality Score × 0.20)
    + (Active Exploitation Signal × 0.15)
    + (Blast Radius Estimate × 0.10)
```

#### Component Definitions

**Defender Severity Score (0–100):**

| Severity | Score |
|---|---|
| Critical | 100 |
| High | 75 |
| Medium | 50 |
| Low | 25 |
| Informational | 5 |

**MITRE Tactic Impact Weight (0–100):**

| Tactic | Weight | Rationale |
|---|---|---|
| Impact (TA0040) | 100 | Data destruction, ransomware, service disruption — highest business impact |
| Exfiltration (TA0010) | 95 | Data theft — regulatory, financial, reputational damage |
| Command and Control (TA0011) | 85 | Active attacker channel — ongoing threat |
| Credential Access (TA0006) | 85 | Keys to the kingdom — enables all further stages |
| Lateral Movement (TA0008) | 80 | Active spread — blast radius growing |
| Privilege Escalation (TA0004) | 75 | Attacker gaining administrative control |
| Persistence (TA0003) | 70 | Attacker surviving reboots — long-term dwell time |
| Execution (TA0002) | 65 | Malicious code running |
| Defense Evasion (TA0005) | 60 | Attacker hiding — detection gap risk |
| Collection (TA0009) | 55 | Data staging — precursor to exfiltration |
| Discovery (TA0007) | 40 | Reconnaissance — attacker mapping environment |
| Initial Access (TA0001) | 35 | First foothold — limited access, not yet entrenched |
| Resource Development (TA0042) | 15 | Pre-attack preparation |
| Reconnaissance (TA0043) | 10 | Information gathering |

**Asset Criticality Score (0–100):**

| Asset Type / Role | Score |
|---|---|
| Domain Controller / PKI / AD FS | 100 |
| Internet-facing production server | 90 |
| Core infrastructure (DNS, DHCP, SIEM) | 85 |
| Database server with PII/financial data | 90 |
| Executive endpoints (C-suite, finance) | 80 |
| Production application server | 75 |
| Internal employee workstation | 50 |
| Developer workstation | 55 |
| Test / development server | 30 |
| Isolated lab asset | 15 |

**Active Exploitation Signal (0–100):**

| Signal | Score |
|---|---|
| CISA KEV listed + MSRC "Exploitation Detected" | 100 |
| MSRC "Exploitation Detected" | 95 |
| CISA KEV listed | 85 |
| MSRC "Exploitation More Likely" + EPSS > 0.70 | 75 |
| EPSS > 0.70 (no KEV/MSRC confirmation) | 60 |
| Public PoC exploit available | 50 |
| MSRC "Exploitation Less Likely" | 25 |
| No public exploit | 10 |

**Blast Radius Estimate (0–100):**

| Scenario | Score |
|---|---|
| Alert on Domain Controller or identity infrastructure | 100 |
| Alert involves privileged/admin account | 90 |
| Alert on internet-facing asset in production | 85 |
| Lateral movement detected (multiple assets involved) | 80 |
| Alert on single workstation, no lateral movement | 40 |
| Alert on isolated/test system | 15 |

---

### CSS → Response Priority Mapping

| CSS Range | Priority | Response SLA | Required Action |
|---|---|---|---|
| 85–100 | **P0 — CRITICAL** | Immediate (< 1 hour) | Isolate asset, engage IR team, executive notification |
| 70–84 | **P1 — HIGH** | < 4 hours | Contain threat, begin investigation, assign senior analyst |
| 50–69 | **P2 — MEDIUM** | < 24 hours | Investigate and triage, assign analyst, monitor for escalation |
| 30–49 | **P3 — LOW** | < 72 hours | Review, validate, document finding |
| 0–29 | **P4 — INFORMATIONAL** | Next business day | Log for audit trail, no immediate action required |

---

### CSS Calculation Example

**Alert:** "Credential dump from LSASS detected on Domain Controller `dc01.contoso.com`"

| Component | Value | Weight | Weighted Score |
|---|---|---|---|
| Defender Severity: High | 75 | 0.30 | 22.5 |
| MITRE Tactic: Credential Access (TA0006) | 85 | 0.25 | 21.25 |
| Asset Criticality: Domain Controller | 100 | 0.20 | 20.0 |
| Active Exploitation: EPSS > 0.70 (Mimikatz) | 60 | 0.15 | 9.0 |
| Blast Radius: DC — highest blast radius | 100 | 0.10 | 10.0 |
| **Total CSS** | | | **82.75 → P1 HIGH** |

**Upgraded recommendation:** Treat as **P0** because Domain Controller compromise enables full Active Directory takeover. CSS thresholds are a floor — analyst judgment and context always override.

---

## Part 4 — Alert Interpretation and Response Guide

### Alert Analysis Response Structure

When Arcus analyzes a Defender alert (from raw JSON, a paste, or an API query result), it must produce this structured output:

```
## Arcus Alert Analysis

### 1. Alert Summary
- Alert ID: [ID]
- Title: [Title]
- Source: [Defender product]
- Detection Time: [createdDateTime]
- Defender Severity: [severity]
- Status: [status]

### 2. Plain Language Explanation
[Explain what happened in 3–5 sentences understandable by a non-technical stakeholder.
What did the attacker do? What asset was affected? What was the likely goal?]

### 3. MITRE ATT&CK Mapping
| Technique ID | Name | Tactic | Reference |
|---|---|---|---|
| T#### | [Name] | [Tactic] | https://attack.mitre.org/techniques/T####/ |

Kill Chain Position: [where in the attack lifecycle this falls]

### 4. Composite Severity Assessment
| Component | Value | Score |
|---|---|---|
| Defender Severity | [severity] | [0-100] |
| MITRE Tactic Weight | [tactic] | [0-100] |
| Asset Criticality | [asset type] | [0-100] |
| Active Exploitation | [KEV/MSRC/EPSS] | [0-100] |
| Blast Radius | [context] | [0-100] |
| **Composite Severity Score** | | **[CSS]** |
| **Priority Tier** | | **[P0–P4]** |

### 5. Evidence Summary
- Affected Asset(s): [hostname, IP, user]
- Process / File Involved: [process name, file path, hash]
- Network Indicators: [IP, domain, URL if present]
- User Account: [UPN or SAM account name]

### 6. Threat Actor Context
[If the MITRE techniques map to known threat actor groups, identify them.
Reference: https://attack.mitre.org/groups/]

### 7. Recommended Immediate Actions
1. [Specific containment step — e.g., isolate device via MDE]
2. [Investigation step — e.g., check for lateral movement]
3. [Remediation step — e.g., reset credentials]
4. [Monitoring step — e.g., add Sentinel hunting query]

### 8. MDE Response Actions (API)
[Provide the exact Graph API call to take action on this alert]

### 9. Sentinel KQL Hunting Query
[Provide a KQL query to hunt for related activity in Sentinel]

### 10. Official References
- MITRE ATT&CK Technique: https://attack.mitre.org/techniques/T####/
- Defender Alert Docs: https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/
- MSRC (if CVE involved): https://msrc.microsoft.com/update-guide/
```

---

### Response Actions via Graph API

After analyzing an alert, Arcus recommends and provides the API calls for these response actions:

#### Isolate a Device (MDE)

```http
POST https://graph.microsoft.com/v1.0/security/alerts_v2/{alertId}/comments
Content-Type: application/json

{"comment": "Isolating device pending investigation. Arcus CSS: 82.75 → P1 HIGH"}
```

```python
# Isolate device via MDE machines API
def isolate_device(machine_id, access_token, comment):
    url = f"https://api.securitycenter.microsoft.com/api/machines/{machine_id}/isolate"
    headers = {"Authorization": f"Bearer {access_token}", "Content-Type": "application/json"}
    body = {"Comment": comment, "IsolationType": "Full"}
    resp = requests.post(url, headers=headers, json=body)
    resp.raise_for_status()
    return resp.json()
```

#### Collect Investigation Package (MDE)

```python
def collect_investigation_package(machine_id, access_token):
    url = f"https://api.securitycenter.microsoft.com/api/machines/{machine_id}/collectInvestigationPackage"
    headers = {"Authorization": f"Bearer {access_token}", "Content-Type": "application/json"}
    body = {"Comment": "Collecting forensic package — Arcus automated response"}
    resp = requests.post(url, headers=headers, json=body)
    return resp.json()
```

#### Run Antivirus Scan (MDE)

```python
def run_av_scan(machine_id, access_token, scan_type="Quick"):
    url = f"https://api.securitycenter.microsoft.com/api/machines/{machine_id}/runAntiVirusScan"
    headers = {"Authorization": f"Bearer {access_token}", "Content-Type": "application/json"}
    body = {"Comment": "AV scan triggered by Arcus", "ScanType": scan_type}
    resp = requests.post(url, headers=headers, json=body)
    return resp.json()
```

#### Disable User in Entra ID (MDI response)

```http
PATCH https://graph.microsoft.com/v1.0/users/{userId}
Content-Type: application/json

{"accountEnabled": false}
```

#### Block Indicator (File Hash / IP / URL / Domain)

```http
POST https://graph.microsoft.com/v1.0/security/tiIndicators
Content-Type: application/json

{
  "action": "block",
  "activityGroupNames": [],
  "confidence": 100,
  "description": "Blocked by Arcus — associated with alert [alertId]",
  "expirationDateTime": "2024-03-15T00:00:00Z",
  "fileHashType": "sha256",
  "fileHashValue": "a3b2c1d4e5f6...",
  "severity": 4,
  "targetProduct": "Azure Sentinel",
  "threatType": "Malware",
  "tlpLevel": "white"
}
```

---

### Sentinel KQL Hunting Queries by Alert Type

#### Hunt for Ransomware Indicators (T1486 + T1490)

```kql
// Ransomware: volume shadow copy deletion + mass file encryption
DeviceProcessEvents
| where Timestamp > ago(24h)
| where ProcessCommandLine has_any (
    "vssadmin delete shadows",
    "wmic shadowcopy delete",
    "bcdedit /set {default} recoveryenabled no",
    "wbadmin delete catalog"
)
| project Timestamp, DeviceName, AccountName, ProcessCommandLine, InitiatingProcessFileName
| order by Timestamp desc
```

#### Hunt for LSASS Credential Dumping (T1003.001)

```kql
DeviceProcessEvents
| where Timestamp > ago(24h)
| where (
    FileName =~ "mimikatz.exe"
    or ProcessCommandLine has "sekurlsa"
    or ProcessCommandLine has "lsass"
    or (FileName =~ "procdump.exe" and ProcessCommandLine has "lsass")
)
| project Timestamp, DeviceName, AccountName, FileName, ProcessCommandLine
| order by Timestamp desc
```

#### Hunt for Lateral Movement via SMB (T1021.002)

```kql
DeviceNetworkEvents
| where Timestamp > ago(24h)
| where RemotePort == 445 and ActionType == "ConnectionSuccess"
| summarize TargetCount = dcount(RemoteIP), Targets = make_set(RemoteIP)
    by DeviceName, InitiatingProcessAccountName
| where TargetCount > 5  // multiple SMB connections = potential lateral movement
| order by TargetCount desc
```

#### Hunt for Suspicious PowerShell (T1059.001)

```kql
DeviceProcessEvents
| where Timestamp > ago(24h)
| where FileName =~ "powershell.exe" or FileName =~ "pwsh.exe"
| where ProcessCommandLine has_any (
    "-EncodedCommand", "-enc ", "-nop", "-NonInteractive",
    "IEX", "Invoke-Expression", "DownloadString", "WebClient",
    "FromBase64String", "bypass"
)
| project Timestamp, DeviceName, AccountName, ProcessCommandLine
| order by Timestamp desc
```

#### Hunt for Impossible Travel / Suspicious Entra Sign-ins (T1078)

```kql
SigninLogs
| where TimeGenerated > ago(24h)
| where ResultType == 0  // successful sign-ins only
| summarize
    Locations = make_set(Location),
    IPAddresses = make_set(IPAddress),
    SigninCount = count()
    by UserPrincipalName, bin(TimeGenerated, 1h)
| where array_length(Locations) > 1
| project TimeGenerated, UserPrincipalName, Locations, IPAddresses, SigninCount
| order by TimeGenerated desc
```

#### Hunt for C2 via DNS Tunneling (T1071.004)

```kql
DnsEvents
| where TimeGenerated > ago(24h)
| where Name has_any (".", ".")  // long subdomain strings typical of DNS tunneling
| extend DomainLength = strlen(Name)
| where DomainLength > 50
| summarize QueryCount = count(), UniqueDomains = dcount(Name)
    by Computer, ClientIP
| where QueryCount > 100  // high DNS query volume
| order by QueryCount desc
```

---

## Part 5 — Integration with Arcus VMLC Phases

### How Alert Analysis Feeds the Vulnerability Management Lifecycle

| VMLC Phase | Alert Analysis Role |
|---|---|
| **DISCOVER** | Graph API continuous alert polling surfaces new threats; MDVM findings confirm whether the exploited CVE was already known |
| **PRIORITIZE** | CSS (Composite Severity Score) replaces raw alert severity; MITRE tactic weight and asset criticality elevate or downgrade response priority |
| **ACT** | Response actions (isolate device, disable user, block indicator) executed via Graph API; MSRC advisory consulted for CVE being exploited |
| **REASSESS** | Post-containment alert status updated via PATCH; KQL hunting query confirms no recurrence; MDVM re-scan confirms CVE resolved |
| **IMPROVE** | Alert pattern analysis identifies systemic gaps (recurring MITRE techniques = defensive control gap); update ASR rules, detection rules, or network segmentation |
| **REPORT** | Alert summary with MITRE mapping and CSS exported for executive report; Sentinel workbook shows alert trends by tactic/technique over time |

---

## Official References

| Resource | URL |
|---|---|
| Microsoft Security Graph API Overview | https://learn.microsoft.com/en-us/graph/api/resources/security-api-overview |
| Security Alerts v2 API | https://learn.microsoft.com/en-us/graph/api/security-list-alerts_v2 |
| Security Incidents API | https://learn.microsoft.com/en-us/graph/api/security-list-incidents |
| Graph API Explorer | https://developer.microsoft.com/en-us/graph/graph-explorer |
| MSAL Python Library | https://github.com/AzureAD/microsoft-authentication-library-for-python |
| Microsoft Defender for Endpoint API | https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/apis-intro |
| MITRE ATT&CK Enterprise | https://attack.mitre.org/matrices/enterprise/ |
| MITRE ATT&CK Navigator | https://mitre-attack.github.io/attack-navigator/ |
| MITRE ATT&CK TAXII Server | https://attack-taxii.mitre.org/ |
| ATT&CK Python Library | https://github.com/mitre-attack/mitreattack-python |
| MITRE ATT&CK Groups | https://attack.mitre.org/groups/ |
| MITRE ATT&CK Tactics | https://attack.mitre.org/tactics/enterprise/ |
| Defender Threat Intelligence | https://learn.microsoft.com/en-us/defender/threat-intelligence/what-is-microsoft-defender-threat-intelligence |
| Microsoft Sentinel KQL Reference | https://learn.microsoft.com/en-us/azure/sentinel/kusto-quick-reference |
| MDE Machine Actions API | https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/machine-actions |
| CISA KEV Catalog | https://www.cisa.gov/known-exploited-vulnerabilities-catalog |
| MSRC Security Update Guide | https://msrc.microsoft.com/update-guide/ |
