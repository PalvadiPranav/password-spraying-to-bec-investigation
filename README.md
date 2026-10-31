
# Password Spraying to BEC: Microsoft 365 Account Takeover Investigation

![Type](https://img.shields.io/badge/type-SOC%20incident%20report-blue)
![Severity](https://img.shields.io/badge/severity-P1%20Critical-red)
![Framework](https://img.shields.io/badge/MITRE%20ATT%26CK-mapped-orange)
![Data](https://img.shields.io/badge/data-synthetic-lightgrey)

A Tier-1 SOC incident report (**CLD-IR-0001**) investigating a simulated Microsoft 365 account takeover at a fictional company, **Cloudora**. It was written for the [MyFirstHack](https://myfirstcyberjob.com) self-directed SOC training. All identities, IP addresses and logs are synthetic.

## Table of Contents
- [Overview](#overview)
- [What's in this repo](#whats-in-this-repo)
- [Incident summary](#incident-summary)
- [Attack timeline](#attack-timeline)
- [Key findings](#key-findings)
- [Indicators of compromise](#indicators-of-compromise)
- [MITRE ATT&CK mapping](#mitre-attck-mapping)
- [Response actions](#response-actions)
- [Investigate it yourself](#investigate-it-yourself)
- [Skills demonstrated](#skills-demonstrated)
- [Disclaimer](#disclaimer)
- [Author](#author)

## Overview
Cloudora's SIEM began flagging many failed Microsoft 365 logins. Correlating sign-in logs with unified audit logs showed a multi-stage attack: a low-and-slow password spraying campaign, two successful account takeovers, a rogue MFA device for persistence, and a hidden inbox rule set up to conceal finance emails ahead of a Business Email Compromise (BEC) attempt.

| Field | Value |
|---|---|
| Report ID | CLD-IR-0001 (ticket CLD-0001) |
| Severity | P1 (Critical) |
| Attack vector | Password spraying against Microsoft 365 authentication |
| Accounts targeted | 20+ |
| Confirmed compromised | 2 (`daniel.reeve`, `priya.nair`) |
| Persistence | Unauthorized MFA device ("Pixel 6") |
| Data at risk | Finance correspondence, SharePoint Online content |
| Outcome | No confirmed exfiltration or fraudulent transaction |

## What's in this repo
```
.
├── CLD-IR-0001_Cloudora_M365_Account_Takeover.pdf   # the full incident report
├── README.md
├── .gitignore
└── resources/
    ├── cloudora_signin_logs.csv    # synthetic sign-in logs (CloudoraSignIn_CL)
    ├── cloudora_audit_logs.csv     # synthetic audit logs (CloudoraAudit_CL)
    ├── kql_query_pack.txt          # KQL queries for the investigation
    └── README.md                   # how to load the data
```

## Incident summary
1. **Aug 8-9, 2026:** Password spraying from three IP addresses in the `102.89.x.x` range, with only a few attempts per account to stay under lockout thresholds.
2. **Aug 10, 03:12 UTC:** First successful login to Daniel Reeve's account, followed by Outlook Web access.
3. **Aug 10, 03:18 UTC:** Attacker registers a new authenticator device ("Pixel 6"), which gives persistence that survives a password reset.
4. **Aug 10, 03:26 UTC:** Azure Portal accessed with the compromised account.
5. **Aug 10, 03:31 UTC:** Inbox rule "RSS Subscriptions" created. It moves mail from `finance@cloudora.io`, or containing "invoice", to the RSS Feeds folder and marks it as read.
6. **Aug 10, 03:47-03:52 UTC:** Second account (Priya Nair) compromised and SharePoint Online accessed.

## Attack timeline
| Time (UTC) | Source | Event |
|---|---|---|
| 2026-08-08 | Sign-in logs | Password spraying begins |
| 2026-08-09 | Sign-in logs | Spraying continues across many accounts |
| 2026-08-10 03:12:05 | Sign-in logs | Successful login, Daniel Reeve |
| 2026-08-10 03:14:30 | Sign-in logs | Outlook Web accessed |
| 2026-08-10 03:18:44 | Audit logs | New MFA device "Pixel 6" registered |
| 2026-08-10 03:26:02 | Sign-in logs | Azure Portal accessed |
| 2026-08-10 03:31:09 | Audit logs | Malicious inbox rule created |
| 2026-08-10 03:47:18 | Sign-in logs | Successful login, Priya Nair |
| 2026-08-10 03:52:40 | Sign-in logs | SharePoint Online accessed |

## Key findings
1. **Account compromise:** Daniel Reeve's account was accessed from `102.89.44.17` after repeated failures (ResultType 50126) followed by a success (ResultType 0).
2. **Password spraying campaign:** Three attacker IPs targeted 20+ accounts over several days. Seventeen accounts saw only failed attempts, and one (`omar.farah`) was cleared as a false positive because its later successful login came from the user's normal location.
3. **Persistence:** An unauthorized authenticator device was added to Daniel Reeve's account. It would keep working after a password reset unless explicitly removed.
4. **BEC preparation:** The inbox rule hid finance-related email from the legitimate owner, a common precursor to payment redirection fraud.

## Indicators of compromise
| Type | Value | Context |
|---|---|---|
| IP address | `102.89.44.17` | Spraying and Daniel Reeve compromise |
| IP address | `102.89.44.23` | Password spraying campaign |
| IP address | `102.89.45.101` | Spraying and Priya Nair compromise |
| Device name | `Pixel 6` | Unauthorized MFA registration |
| Inbox rule | `RSS Subscriptions` | Malicious mailbox rule |
| Email address | `finance@cloudora.io` | Targeted by the inbox rule |
| Keyword | `invoice` | Used by the inbox rule |

## MITRE ATT&CK mapping
| Tactic | ID | Technique | Evidence |
|---|---|---|---|
| Credential Access | T1110.003 | Password Spraying | Finding 2 |
| Initial Access | T1078 | Valid Accounts | Finding 1 |
| Persistence | T1098.005 | Additional Cloud Credentials (device registration) | Finding 3 |
| Defense Evasion | T1564.008 | Hide Artifacts: Email Hiding Rules | Finding 4 |

## Response actions
**Containment:** force sign-out of both compromised accounts, block the three attacker IPs at the identity provider, suspend the malicious inbox rule.

**Eradication:** remove the "Pixel 6" device, delete the inbox rule, reset passwords, revoke refresh tokens and sessions, and check for extra app registrations, delegated permissions or forwarding rules.

**Recovery and hardening:** review Priya Nair's SharePoint activity, have finance verify invoices processed during the compromise window, and add detection for low-and-slow spraying, new MFA registrations and suspicious inbox rules. Moving privileged and finance accounts to phishing-resistant MFA (FIDO2) is also recommended.

> The actions above are documented recommendations. The investigation was done in a training environment and nothing was executed against production systems.

## Investigate it yourself
The `resources/` folder lets you run the same investigation and write your own report.

1. Open [Azure Data Explorer](https://dataexplorer.azure.com) and create a free cluster.
2. Ingest the two CSVs as tables named exactly `CloudoraSignIn_CL` and `CloudoraAudit_CL`.
3. Set `TimeGenerated` to datetime and `ResultType` to **string** at ingestion.
4. Work through `kql_query_pack.txt`, then document your findings.

Example query, which finds source IPs spraying many accounts:
```kql
CloudoraSignIn_CL
| where ResultType == "50126"
| summarize FailedAttempts = count(), TargetedAccounts = dcount(UserPrincipalName)
    by IPAddress, bin(TimeGenerated, 1d)
| where TargetedAccounts > 5
| order by FailedAttempts desc
```
See the report appendix for the full set of queries.

## Skills demonstrated
- Log analysis and threat hunting with KQL
- Correlating sign-in and audit telemetry to rebuild an attack chain
- IOC extraction and false-positive triage
- MITRE ATT&CK mapping
- Containment, eradication and recovery planning
- Writing clear incident reports for technical and executive readers

## Disclaimer
Cloudora is a fictional organization and all data is synthetic. This repository is for education and portfolio purposes only and does not represent employment with any company. The exercise itself was created by the MyFirstHack community; the report and its analysis are my own work.

## Author
**P.S.V.S. Pranav** | CEH v13 | 
[GitHub](https://github.com/PalvadiPranav)
