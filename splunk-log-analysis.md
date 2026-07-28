
# Splunk Log Analysis Lab — Windows Event Log Monitoring

## Overview
Set up a small log monitoring pipeline using Splunk Enterprise and the Splunk Universal Forwarder, then analyzed real Windows Event Logs to investigate login activity — including failed login attempts and successful logins — on a monitored virtual machine.

## Environment
- **Splunk Enterprise:** Physical Windows 11 laptop (search/analysis)
- **Splunk Universal Forwarder:** Windows 10 VM (VMware) — monitored machine
- **Logs collected:** Security, Application, System, Setup, Forwarded Events
- **Connection:** Forwarder → physical machine IP on port 9997

## Tools Used
Splunk Enterprise, Splunk Universal Forwarder, VMware, SPL (Splunk Search Processing Language)

## Steps Performed
1. Installed and connected Splunk Enterprise and Universal Forwarder across host and VM
2. Verified log ingestion with `index=*`
3. Filtered to security events: `index=* source="WinEventLog:Security"`
4. Searched failed logins: `index=* EventCode=4625`
5. Counted failed logins by account: `index=* EventCode=4625 | stats count by Account_Name`
6. Charted failed logins over time: `index=* EventCode=4625 | timechart count by Account_Name`
7. Compared against successful logins: `index=* EventCode=4624 | stats count by Account_Name`

## Evidence
![Failed login search results](./screenshots/failed-logins.png)
*Failed login attempts (Event ID 4625)*

![Failed logins by account](./screenshots/failed-logins-by-accountname.png)
*Failed login count grouped by account*

![Failed logins timechart](./screenshots/chart-showing-spikes-by-date.png)
*Failed login attempts over time*

![Successful logins by account](./screenshots/successful-login-by-accountname.png)
*Successful login events by account*

## Key Findings
- Failed logins (Event ID 4625) were recorded against three local test accounts, all originating from `127.0.0.1`, with "Bad username or password" as the consistent failure reason.
- Failed attempts weren't evenly distributed — certain dates showed clusters of 4-7 failures, a pattern that would typically prompt closer review in a live environment.
- Successful logins (Event ID 4624) included genuine user activity alongside built-in Windows system/service accounts (SYSTEM, DWM, UMFD, LOCAL/NETWORK SERVICE) and machine accounts — all normal background activity once identified correctly.

## Insights
- Clustered failed-login patterns are exactly what a SOC analyst would investigate further for signs of brute-force activity — the same `timechart` method applies directly to spotting this in a live alert.
- Distinguishing real user logins from Windows system/service accounts is essential for reducing false positives during alert triage — treating every login event as suspicious creates unnecessary noise.
- In a live scenario, the natural next step after spotting repeated failed logins would be checking whether a successful login followed shortly after — a common indicator of an eventually successful brute-force attempt.

## What I Learned
Setting up the forwarder-to-indexer pipeline gave practical exposure to how logs move from an endpoint to a central SIEM — not just querying pre-loaded data. Learning to separate genuine login activity from normal Windows background processes was the most valuable takeaway, since accurate triage depends on filtering out expected noise.

## MITRE ATT&CK Mapping

This lab's detection scenarios map to the following MITRE ATT&CK techniques:

| Technique ID | Technique Name | Relevance to this Lab |
|---|---|---|
| T1078 | Valid Accounts | Detected use of legitimate credentials to access systems in ways that deviated from normal behavior, consistent with adversaries leveraging valid accounts to blend in with normal activity. |
| T1190 | Exploit Public-Facing Application | Identified log activity consistent with an attacker targeting an internet-facing application/service as an initial access vector. |

**Reference:** [MITRE ATT&CK Matrix](https://attack.mitre.org/)

## Skills Demonstrated
`Splunk Enterprise` `Splunk Universal Forwarder` `SPL` `Windows Event Log Analysis` `VMware` `Log Correlation` `False Positive Reduction`
