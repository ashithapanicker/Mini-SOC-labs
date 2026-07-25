# SOC176 - RDP Brute Force Detected (LetsDefend)

**MITRE ATT&CK:** T1110 – Brute Force (T1110.001 – Password Guessing)

## Overview
This is my first alert investigation on LetsDefend's SOC Simulator. The alert flagged repeated RDP login failures on a host called "Matthew" from an external IP address. I picked this one first because it's a classic SOC scenario and ties directly into brute-force concepts I'd already covered in theory.

## Objective
Investigate the alert, determine whether the activity was malicious, check if any login attempt succeeded, and submit a verdict with a recommended action.

## Approach / Environment / Tools Used
- Platform: LetsDefend SOC Simulator
- Tools used within the platform: Monitoring (alert queue), Log Management (event search), Threat Intel lookup (VirusTotal)
- Alert type: RDP Brute Force Detected (SOC176)

## Steps Performed
1. Took ownership of the alert from the Monitoring queue.
2. Reviewed the alert details:
   - Source IP: 218.92.0.56
   - Destination IP / Hostname: 172.16.17.148 / "Matthew"
   - Protocol: RDP (port 3389)
   - Firewall Action: Allowed
   - Trigger reason: login failures from a single source using different, non-existing accounts
3. Checked the source IP's reputation on VirusTotal — flagged by 8 out of 91 security vendors.
4. Searched Log Management, filtering by destination_address = 172.16.17.148, narrowed to the alert's time window.
5. Found 30 total connection events from the same source IP to the same destination on port 3389.
6. Expanded individual log entries and found two events in detail — both EventID 4625 (login failure), with usernames "guest" and "admin," error code 0xC000006D (unknown username or bad password).
7. Based on checking a sample of entries, initially concluded all attempts failed and submitted a "True Positive - contained" verdict, recommending the source IP be blocked.

## Evidence
![Alert Details](./screenshots/letsdefend-alert-details.png)

ALERT DETAILS 


![VirusTotal Check](./screenshots/virustotal-check.png)

VIRUSTOTAL CHECK


![Log Search Results](./screenshots/letsdefend-log-search.png)

LOG SEARCH RESULTS 


![Successful Login Found](./screenshots/letsdefend-successful-login.png)

 SUCCESSFUL LOGIN

## Key Findings
- Attacker IP 218.92.0.56 made 30 RDP connection attempts against host "Matthew" over a short window, cycling through different usernames.
- Two verified failed logins used non-existent usernames ("guest," "admin").
- **What I initially missed:** I only checked a couple of events on the first page of results and assumed the rest followed the same failed pattern. LetsDefend's grading flagged that the brute force actually succeeded — one of the later events (found on page 3 of the log search) showed a successful login (EventID 4624) using the username "Matthew" — the same as the hostname, which in hindsight was a clue I should have checked for specifically, since attackers often try usernames based on the hostname.

## Insights / Analysis
This alert taught me that in brute-force investigations, sampling a few log entries isn't enough — the whole point of brute force is that it usually succeeds eventually, and that success is often buried among many failures rather than at the start. I also learned to pay attention to hostname-based username guesses as a specific pattern to check for, not just generic ones like "admin" or "guest."

## What I Learned
- EventID 4625 = failed logon, EventID 4624 = successful logon — useful to have memorized rather than looked up each time.
- Always review every log entry for the destination in a brute-force case, not a sample — the compromise is often near the end of the attempts, not the beginning.
- A "Firewall Action: Allowed" on an alert doesn't mean the login succeeded — it just means the traffic reached the host; the actual outcome has to be verified in the authentication logs.
- Got this one partially wrong on my first pass (missed the successful login), went back into the logs, found it myself, and corrected my understanding before finalizing my notes here.

## Skills Demonstrated
Alert triage and ownership, log correlation across multiple events, threat intelligence lookups (VirusTotal), Windows authentication event analysis (4624/4625), brute-force attack pattern recognition, incident verdict documentation, self-review and correction of an initial analysis.
