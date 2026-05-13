# RMM Attack Simulation and Defense Lab

**Author:** Gbenga Abraham  
**Tenant:** ea{xxx}eit.com  
**Environment:** Microsoft Azure | Microsoft 365 E5 | Microsoft Sentinel  
**Exercise type:** Purple Team - Full attack simulation with detection and response  
**Date:** May 2026

---

## What This Project Is About

Most security portfolios show tool configuration screenshots. This one shows the full picture.

This lab simulates a real-world phishing-to-exfiltration attack using a commercial RMM tool against a corporate endpoint with zero security controls. Every event is captured with raw telemetry, mapped to MITRE ATT&CK, and used to build targeted detection rules. Security controls are then deployed and the attack is repeated to demonstrate measurable improvement.

The exercise covers initial access, execution, persistence, C2 communication, credential access, data staging, exfiltration, detection engineering, and incident response — end to end.

> **Target roles:** SOC Analyst | Incident Responder | IAM Analyst



---

## Lab Environment

| Component | Detail |
|---|---|
| Cloud platform | Microsoft Azure |
| Identity provider | Microsoft Entra ID |
| Endpoint | Windows 11 VM - soclab |
| License | Microsoft 365 E5 |
| SIEM | Microsoft Sentinel |
| EDR | Microsoft Defender for Endpoint |
| Email security | Microsoft Defender for Office 365 Plan 2 |
| MDM | Microsoft Intune |
| RMM tool | Atera (trial) |
| Attacker mailbox | soclabteXXXX@outlook.com |
| Victim mailbox | test@eagXXXXreit.com |

---

## Phase 1 — Attack Simulation (Run 1 — No Controls)

### Overview

The attacker registered a free Atera RMM trial, generated an agent installer, renamed it to AdobeFile.msi to appear legitimate, and delivered it via a spearphishing email. The victim opened the email, downloaded and executed the file, and the attacker established persistent remote access within minutes.

**Total dwell time before exfiltration: approximately 5 hours. Zero alerts fired.**

---

### Step 1 — Phishing Email Crafted and Sent

The attacker composed a spearphishing email impersonating a project coordinator named Alex Morgan, with the subject "Document Review — Urgent" and a malicious link embedded in the body.

[![Attacker building phishing email](<img width="800" alt="image" src="https://github.com/user-attachments/assets/5893145d-c662-495b-aae1-8e48b2bd7405" />
)

---

### Step 2 — Email Delivered to Focused Inbox

The email landed directly in the victim's Focused inbox with no filtering, no quarantine, and no warning. The external sender impersonating a colleague was not flagged by any control.


---
<img width="800" alt="image" src="https://github.com/user-attachments/assets/9a3f185f-1219-4bfd-ae63-1cf172e6be9e" />

---

---

### Step 3 — AdobeFile.msi Downloaded and Executed

The victim downloaded and ran the installer. Windows configured AteraAgent silently in the background. The filename masqueraded as an Adobe file to avoid suspicion.

---
<img width="864"  alt="AdobeFile Installing" src="https://github.com/user-attachments/assets/8390d609-0707-4cd9-855f-bf7c7cbac973" />


---

### Step 4 — Attacker Gains Remote Access via Atera

The Atera portal showed the soclab device as Online within minutes. The attacker had full remote visibility and control of the victim desktop.

---

<img width="800" alt="image" src="https://github.com/user-attachments/assets/a8297aab-71b5-48a4-9255-45647e78fbd2" />


---

### Step 5 - Victim Desktop and Sensitive Data Visible

The attacker viewed the victim desktop remotely showing decoy folders: Payroll, Financial Records, HR, Employee Records, Credit Cards, Health Records, Crypto Wallet Keys, and Saved Logins.

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/6bc502a6-d05d-4e09-a9c2-a5b9a3bb7e21" />


---

### Step 6 — Data Staged and Compressed

Sensitive folders were compressed into a ZIP archive on the Desktop. MDE captured the FileCreated event for HR.zip and the FileRenamed event when it became EXPORT FILE.zip.

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/5810c3f0-18c5-4b77-8f5f-f5973eb93cf2" />

---

### Step 7 — Data Exfiltrated to External Platform

EXPORT FILE.zip was uploaded to an external file sharing platform via the browser. The upload was invisible in DeviceNetworkEvents because browser HTTPS traffic on port 443 blends with normal web activity — a critical detection gap documented in the lessons learned.

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/6cb1b62d-4a6b-4242-86ff-d316821438b6" />


---

### Attack Timeline

| Time | Event | MITRE TTP |
|---|---|---|
| 9:30 PM | Phishing email composed | T1566.001 |
| 9:52 PM | Email lands in Focused inbox | T1566.001 |
| 10:02 PM | AdobeFile.msi downloaded and executed | T1204.002, T1036.004 |
| 5:04:59 PM | msiexec spawns rundll32 to drop installer DLL | T1218.011 |
| 5:05:39 PM | AteraAgent.exe written to Program Files | T1059 |
| 5:05:45 PM | Atera initialised with unique agent ID | T1219 |
| 5:06:09 PM | AteraAgent registered as Windows service | T1543.003 |
| 5:06:16 PM | First C2 beacon to agent-api.atera.com | T1219, T1573 |
| 5:06:20 PM | PubNub C2 command channel established | T1071.001 |
| 5:06:20 PM | DPAPI credential store accessed | T1555.004 |
| 5:08:26 PM | Azure ServiceBus heartbeat channel active | T1573.002 |
| 5:11:27 PM | Installer artefact deleted to cover tracks | T1070.004 |
| 5:30 PM | Splashtop deployed as second RAT | T1219 |
| 9:21:16 PM | Encoded PowerShell executed remotely | T1059.001, T1140 |
| 10:10 PM | HR.zip created and renamed EXPORT FILE.zip | T1074.001 |
| 10:15 PM | EXPORT FILE.zip uploaded externally | T1048 |

---

### Telemetry Evidence

**Atera C2 beaconing confirmed in DeviceNetworkEvents**

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/16380dbc-1d6d-4421-bc05-b36d10afdf7e" />

--


**NTLM logon events with no MFA challenge**

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/3daac699-0ecb-426e-b22c-308f7d087d5e" />

---


**ZIP file staging confirmed in DeviceFileEvents**

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/5dfb4a32-efda-42d7-987e-0d672e944631" />

---

### Security Gaps — Run 1 Summary

| Layer | Gap | Impact |
|---|---|---|
| Email | No Safe Attachments or Safe Links | MSI delivered and executed freely |
| Email | No anti-phishing policy | External impersonation landed in inbox |
| Identity | No MFA on employee account | No second factor required |
| Identity | No Conditional Access | Any device, any location, any time |
| Identity | Legacy auth not blocked | 11 NTLM logons went unchallenged |
| Endpoint | No ASR rules | rundll32 proxy execution undetected |
| Endpoint | No App Control | Unsigned binaries ran freely |
| Endpoint | No device compliance | Unmanaged device had full access |
| Data | No DLP policy | ZIP upload completely invisible |
| Detection | No Sentinel analytics rules | 5 hours of activity, zero alerts |

---

## MITRE ATT&CK Mapping

| Tactic | Technique | ID | Evidence |
|---|---|---|---|
| Initial Access | Spearphishing attachment | T1566.001 | AdobeFile.msi via email |
| Initial Access | Spearphishing link | T1566.002 | Malicious URL in email body |
| Execution | Malicious file | T1204.002 | Victim executed MSI from Downloads |
| Execution | PowerShell | T1059.001 | Encoded command via agentpackagestremote.exe |
| Execution | Rundll32 | T1218.011 | msiexec spawned rundll32 as proxy |
| Persistence | Windows service | T1543.003 | AteraAgent service at 5:06:09 PM |
| Persistence | Remote access software | T1219 | Atera plus Splashtop dual RAT |
| Defense Evasion | Masquerading | T1036.004 | AdobeFile.msi filename |
| Defense Evasion | Deobfuscation | T1140 | Base64 encoded PowerShell |
| Defense Evasion | Indicator removal | T1070.004 | 8-0-11.exe deleted post-install |
| Credential Access | DPAPI | T1555.004 | ateraagent.exe accessed credential store |
| Credential Access | NTLM | T1550.002 | 11 network logons without MFA |
| Command and Control | Remote access software | T1219 | Four Atera C2 domains |
| Command and Control | Encrypted channel | T1573 | TLS 1.2 on all beacons |
| Command and Control | Web protocols | T1071.001 | PubNub HTTPS command channel |
| Collection | Local data staging | T1074.001 | HR.zip on Desktop |
| Collection | Archive collected data | T1560 | Compressed to ZIP |
| Exfiltration | Alternative protocol | T1048 | EXPORT FILE.zip uploaded |
| Exfiltration | Web service | T1567 | Public file share destination |

---

## Phase 2 - Security Controls Deployed (Run 2)

### Email Gateway — Microsoft Defender for Office 365

**Anti-malware policy**

Blocked file types: exe, msi, bat, cmd, vbs, js, ps1, hta, lnk, reg, jar. Action set to quarantine. Zero-hour auto purge enabled to retroactively quarantine emails already delivered if a threat is later identified.

---

<img width="800" alt="image" src="https://github.com/user-attachments/assets/20357c2b-02ac-4e54-9e32-f7ff9c8d94af" />


---

**Safe Attachments**

Action: Block. Quarantine policy: AdminOnlyAccessPolicy — users cannot self-release. Sandbox detonation holds delivery until verdict is confirmed.

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/f33fa2d8-5165-49ba-993b-e5eff85e1b17" />

---

**Safe Links**

Active across email, Teams, and Office 365 apps. Real-time scanning before delivery. Users cannot click through to the original URL. Click tracking feeds UrlClickEvents for detection rules.

---

<img width="2220" height="1256" alt="image" src="https://github.com/user-attachments/assets/0eb6fe16-f470-4ebd-ba52-bb2b879084a3" />


---

**Safe Links validated**

Hovering over the phishing link showed the URL rewritten through can01.safelinks.protection.outlook.com — proof the policy is active and scanning every click before the user reaches the destination.

---


<img width="800"  alt="image" src="https://github.com/user-attachments/assets/e7839272-f31d-4cda-9bb3-9f8288103a67" />

---

**Anti-phishing**

Threshold: 3 — Most Aggressive. Mailbox intelligence on. Spoof intelligence on. Priority 1 — evaluates before any other rule.

---
<img width="800"  alt="image" src="https://github.com/user-attachments/assets/2a827713-0346-41d7-9248-447dbe53fe0e" />

---

---

### Identity — Microsoft Entra Conditional Access

Eleven policies deployed, all enabled and active.

[![Conditional Access policies list](screenshots/07-identity-controls/conditional-access-policies.png)](screenshots/07-identity-controls/conditional-access-policies.png)

**Device registration MFA validated**

CA3 fired when the VM attempted Entra join — Microsoft Authenticator number matching was required before registration proceeded. A compromised account cannot silently register a rogue device.

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/07f18b13-c988-4b5a-b2f9-34f501dcd05f" />

---

**Device compliance confirmed**

Soclab VM showed Compliant in Intune after enrollment - managed by Intune, corporate ownership, Windows 10.0.26200.8246.

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/1731adfc-41c0-471f-913b-428f09cf05e4" />


---

### Endpoint — Intune and Microsoft Defender

**Attack Surface Reduction Rules — all in Block mode**

All rules deployed to All Devices group. Enforced via both MDM and MicrosoftSense channels after enabling MDE security settings management.

---

<img width="800" alt="image" src="https://github.com/user-attachments/assets/ebb4146b-84ce-4c94-8901-9e2754fb4cd6" />

---

**ASR rules confirmed on device**

PowerShell on the VM confirmed all rule IDs were present and active on the endpoint.

---


<img width="800" alt="image" src="https://github.com/user-attachments/assets/5f4bb547-e0d8-47cf-8a94-b7282d8ce527" />

---

**Application Control for Business — WDAC**

Built-in controls. Audit mode disabled, full enforcement. Unsigned executables from user-writable paths blocked at kernel level before process creation.


---
<img width="800" alt="image" src="https://github.com/user-attachments/assets/02b4c0a3-c3a5-4cb0-9856-0e74ce3faea4" />


---

**WDAC validated**

Custom unsigned test binary placed on Desktop. Execution denied — "Windows cannot access the specified device, path, or file." Blocked at kernel level before the process started.

---

<img width="800" alt="image" src="https://github.com/user-attachments/assets/3caa3dc3-00d8-4db0-8476-a52143ffa00a" />


---

### Run 2 Result: AdobeFile.msi Blocked

Two independent controls fired simultaneously against the same file:

Windows Installer policy: "The system administrator has set policies to prevent this installation."  
ASR rule: "This content is blocked, your administrator is not allowing you to access content from AdobeFile.msi."

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/860adf52-897b-44ff-8e76-7776d0d40eff" />

---

Run 1 — silent install completed in 3 minutes with zero alerts.  
Run 2 — blocked before execution by two independent controls simultaneously.

---

## Phase 3 — Detection Engineering

### Custom Sentinel Analytics Rules

Seven scheduled analytics rules authored and deployed in Microsoft Sentinel, each mapped to specific MITRE techniques observed in Run 1 telemetry.

[![Sentinel analytics rules list](screenshots/09-detection-rules/sentinel-analytics-rules-list.png)](screenshots/09-detection-rules/sentinel-analytics-rules-list.png)

Full KQL for all seven rules is available in the [detection-rules/](detection-rules/) folder.

| Rule | Tactic | Severity | Frequency |
|---|---|---|---|
| RMM tool downloaded via browser | Initial Access | High | Every 5 min |
| RMM execution blocked by ASR or App Control | Execution | High | Every 5 min |
| RMM tool installed as Windows service | Persistence | High | Every 5 min |
| Unauthorised RMM process execution | C2 | High | Every 5 min |
| RMM C2 outbound beacon | C2 | High | Every 5 min |
| MSI executed from user writable path | Execution | High | Every 5 min |
| Full RMM kill chain correlation | Multiple | Critical | Every 5 min |

---

### Incidents Generated in Production

Rule 6 generated three High severity incidents during Run 2 testing, correctly categorised as Execution and auto-tagged with MITRE techniques T1204, T1036, T1204.002, and T1036.004.

---

<img width="800"  alt="image" src="https://github.com/user-attachments/assets/6f5dd64b-693f-42de-8a17-0243eea73a42" />



---

## Phase 4 — Incident Response

### Incident ID 96 — Investigated and Resolved

---
<img width="800" alt="image" src="https://github.com/user-attachments/assets/a430c556-dfb9-4750-a63d-504dccbcc51a" />

---

| Field | Value |
|---|---|
| Incident ID | 96 |
| Severity | High |
| First activity | May 12, 2026, 1:18 PM |
| Detection source | Microsoft Sentinel scheduled rule |
| MITRE techniques | T1204, T1036, T1204.002, T1036.004 |
| Assigned to | GbenXXXXX@XXXreIT |
| Status | Resolved |
| Classification | True Positive - Security Testing |
| Resolution | Confirmed test activity - AdobeFile.msi execution attempt blocked by ASR policy |

---

### IR Playbook — RMM-Based Phishing Attack

**Containment — first 15 minutes**

Isolate device in MDE to cut C2 while preserving forensics. Revoke all active Entra ID sessions. Reset account credentials. Block attacker sender domain via Exchange mail flow rules. Disable compromised account pending investigation.

**Eradication**

Kill all RMM processes. Stop and delete AteraAgent Windows service. Remove installation directories and ProgramData folders. Clean registry keys. Search for and remove scheduled tasks. Purge phishing email from all mailboxes using Content Search hard delete.

**Recovery**

Restore device from clean pre-attack snapshot. Re-enable account only after MFA and CA policies are confirmed active. Monitor for 72 hours. Validate all controls before returning to production.

---

### Lessons Learned

| Finding | Severity | Remediation |
|---|---|---|
| No MFA on employee accounts | Critical | 11 CA policies deployed — MFA enforced |
| Email gateway not configured | Critical | Anti-malware, Safe Attachments, Safe Links, Anti-phishing deployed |
| No ASR rules | High | All rules in Block mode via Intune |
| No device compliance | High | CA5 — Require Compliant Device active |
| Legacy NTLM accepted | High | CA8 — Block Legacy Auth active |
| Signed RMM tools bypass App Control | Medium | Proactive Remediation script as compensating control |
| Browser exfiltration invisible | Medium | Purview DLP recommended as next control |
| Link-only phishing bypasses gateway | Medium | URL block list and external sender banner recommended |

---

## Skills Demonstrated

**SOC Analyst** — Threat hunting with KQL across five MDE tables. Raw telemetry analysis from a 9,000-event MDE export. Kill chain reconstruction with precise timestamps. Custom Sentinel analytics rule authoring, deployment, and validation. Incident triage and structured resolution.

**Incident Responder** — Full IR lifecycle following NIST SP 800-61. Attack timeline reconstruction from raw telemetry. MITRE ATT&CK mapping across 19 techniques and 8 tactics. Containment, eradication, and recovery documented. Gap analysis with prioritised remediation.

**IAM Analyst** — Conditional Access design and deployment across 11 policies. Entra ID Identity Protection configuration. Device registration workflow secured with MFA. Intune enrollment and compliance policy. Least privilege enforcement via Windows Installer restriction, UAC policy, and WDAC.

---

## Technologies Used

Microsoft Azure, Microsoft Entra ID, Microsoft Intune, Microsoft Defender for Endpoint, Microsoft Defender for Office 365 Plan 2, Microsoft Sentinel, Microsoft Defender XDR, Atera RMM, KQL, PowerShell, Windows Defender Application Control, Attack Surface Reduction Rules, Conditional Access, Safe Attachments, Safe Links, Zero-Hour Auto Purge, Microsoft Authenticator, Azure Bastion, Just-in-Time VM Access.

---

