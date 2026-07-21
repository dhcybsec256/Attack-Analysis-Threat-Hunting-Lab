## Overview

This repository documents a self-directed practical project simulating real-world attack analysis and incident response activities across six exercises: adversary behaviour mapping using the MITRE ATT&CK framework, phishing email forensics, network packet capture analysis, Intrusion Detection System (IDS) deployment with custom rule authoring, and two proactive threat hunting labs.

All investigations were conducted inside isolated virtual machines using fictional, self-created scenarios and data, in line with standard cybersecurity operations practices. The project is structured to mirror the practical workflow of a SOC analyst or incident responder: understanding adversary tactics through structured frameworks, identifying malicious artifacts in raw data (emails, packet captures, logs), configuring and tuning detection tooling, and conducting evidence-based threat hunting to uncover and document full attack chains.

## Disclaimer

This report documents simulated attack scenarios created solely for educational and defensive research purposes. All incidents, IP addresses, domains, organisations, and individuals referenced in this project are fictional and do not represent real entities or events. All testing and analysis was conducted in isolated, non-production environments with no impact to external systems or networks. The techniques, detection rules, and IOCs documented within this project are provided for the purpose of improving detection engineering, threat hunting capability, and incident response skill, and must not be used to develop, deploy, or facilitate malicious activity. This report does not constitute legal advice.

## Objectives

- Map simulated adversary behaviour to the MITRE ATT&CK framework to build structured, industry-standard incident narratives.
- Identify malicious indicators in phishing emails through manual header analysis and sandbox tooling.
- Analyse network packet captures to detect and validate multiple classes of network attacks.
- Deploy and configure an IDS (Suricata) and author custom detection rules for command-and-control (C2) traffic, data exfiltration, and DNS tunnelling.
- Conduct proactive threat hunting against Suricata alert logs using command-line tooling (jq) to uncover multi-stage intrusions.
- Document all findings, hypotheses, and conclusions in accordance with incident response and threat intelligence best practices.

## Analysis Environment

- Phishing analysis was conducted using a text editor for manual header inspection, combined with the PhishTool sandbox platform and VirusTotal for indicator validation.
- Network packet analysis was performed offline in Wireshark against static pcap files, with no live traffic generation against production systems.
- IDS deployment and rule testing was performed on a Kali Linux virtual machine running Suricata in IDS mode, with all test traffic generated and contained locally.
- Threat hunting labs were conducted against pre-generated Suricata alert logs (JSON format), parsed and analysed using jq on the same isolated Kali Linux environment.

## Tools Used

| Tool | Purpose |
|---|---|
| MITRE ATT&CK Framework | Adversary tactic, technique, and procedure (TTP) mapping |
| Wireshark | Packet capture analysis and protocol-level investigation |
| Suricata | IDS deployment, traffic monitoring, and custom rule detection |
| jq | JSON log parsing and command-line threat hunting |
| PhishTool | Phishing email sandbox and header analysis |
| VirusTotal | URL, file, and hash reputation analysis |
| OSINT tools (e.g. whatismyipaddress.com) | IP attribution and geolocation |
| Kali Linux | IDS host environment |

## Project Scope

The project covers six self-contained exercises, each simulating a distinct stage or discipline within attack analysis and incident response.

### 1. MITRE ATT&CK Adversary Behaviour Mapping

A fictional university phishing incident was constructed and manually mapped to the MITRE ATT&CK Enterprise Matrix, producing a full 10-stage technique breakdown covering initial access through spear phishing, credential theft via a spoofed login portal, lateral movement using built-in administrative tools, data collection and archiving, C2 communication, and exfiltration to external cloud storage. Ten tactics were mapped end-to-end, including Initial Access (T1566.002), Execution (T1204, T1059), Persistence (T1078), Defense Evasion (T1070.004), Credential Access (T1056.003), Discovery (T1083), Lateral Movement (T1021), Collection (T1560.001), Command and Control (T1071.001), and Exfiltration (T1567.002).

### 2. Phishing Email Forensics

Two phishing email samples were analysed using two independent methods:

- **Manual header analysis** in a text editor, examining Message-ID, Sender, Reply-To, Return-Path, and SPF/DKIM/DMARC authentication results. The first sample showed domain spoofing between the claimed sender (otto.de) and the actual infrastructure (firiri.shop), a failed DMARC check, and malicious hyperlinks shortened via t.co that were confirmed as phishing and malware-carrying by multiple VirusTotal vendors.
- **Sandbox analysis** using PhishTool on a second sample, an impersonation scam posing as a Brazilian bank notification. This sample passed SPF/DKIM/DMARC but showed a mismatched display name versus sender address, and carried a PDF attachment confirmed as trojan malware by 18 of 64 VirusTotal vendors.

Both samples were independently verdicted as phishing, with supporting evidence documented for each authentication check, hyperlink, and attachment.

### 3. Network Packet Capture Analysis (Two Labs)

Two pcap files were triaged in Wireshark, starting from the Conversations window to identify anomalous traffic volumes, then isolated using targeted display filters (`ip.addr`, `tcp.flags`, `http.request.method`) to confirm attack type. Six distinct attack patterns were identified and validated using protocol-level evidence:

- **ICMP Flood** — automated ICMP Type 11 (TTL Exceeded) and Type 3 (Destination Unreachable) error messages sent at an inhuman rate to exhaust target resources.
- **TCP ACK Flood / Port Scan** — an automated scanner using incrementing source ports, hardcoded TCP sequence numbers, and fixed window sizes; confirmed via handshake analysis that no open ports were found.
- **HTTP GET Flood** — 180 GET requests in roughly 10 seconds, each opening a fresh TCP connection and forcibly reset with RST rather than a clean FIN, consistent with automated flooding tooling.
- **HTTP POST Flood** — 40 POST requests with empty bodies, hardcoded User-Agent strings, and clean FIN/ACK teardown designed to mimic legitimate traffic while exhausting server processing.
- **TCP SYN Flood** — network-layer flood cycling every second alongside the GET/POST floods, characteristic of a multi-layer coordinated DDoS.
- **CDN Cache-Bypass HTTP Flood** — 284 HTTP GET requests in 9 seconds using `Cache-Control: no-cache` and `Pragma: no-cache` headers to force each request past Cloudflare's caching layer directly onto the origin server, confirmed via `CF-Cache-Status: DYNAMIC` in the response headers.

### 4. Suricata IDS Deployment and Custom Rule Development

Suricata was installed and configured in IDS mode on a Kali Linux virtual machine, including HOME_NET validation, custom rule file creation, and resolution of configuration issues (missing rules file, incorrect default rule path). Three custom detection rules were authored, justified, and live-tested against generated traffic:

- **Rule 1 — HTTP-based C2 detection:** flags HTTP POST requests with base64-encoded form-urlencoded payloads, rate-limited to catch repeated beaconing behaviour rather than single benign requests.
- **Rule 2 — HTTPS-based data exfiltration detection:** flags large POST requests (greater than 1000 bytes) containing base64-encoded content, thresholded to distinguish bulk exfiltration from normal uploads.
- **Rule 3 — DNS tunnelling detection:** uses a regex to flag DNS queries with subdomains 40+ characters long, consistent with base64-encoded tunnelled data, plus an additional rule to catch high-frequency short-query tunnelling that evades the long-domain threshold by chunking data across many queries.

All three rules (plus the optional fourth) were validated by generating matching traffic with `curl` and `dig` and confirming alerts in `fast.log`.

### 5. Proactive Threat Hunting Lab 1 (QakBot to Gozi Banking Trojan)

A pcap file was converted into structured JSON alert and event logs using Suricata, then investigated using jq. Four hypotheses were formed and tested against the alert signatures (malware delivery via EXE, QakBot C2 beaconing, Gozi C2 beaconing, and DNS manipulation). The investigation reconstructed a complete infection chain:

1. A malicious EXE masquerading as a PNG file was delivered over unencrypted HTTP from a compromised WordPress site.
2. QakBot began encrypted C2 beaconing over port 443 at regular one-second intervals to a residential IP, consistent with a hijacked home network being used as a C2 relay.
3. Suspicious DNS traffic with unparseable NULL query fields was observed throughout the infection.
4. QakBot deployed Gozi as a second-stage payload, which established its own C2 channel to two fallback servers hosted on legitimate commercial cloud infrastructure (Edgecast) to blend in with normal traffic and evade blacklists.

The full attack timeline was mapped to MITRE ATT&CK across Initial Access, Execution, Persistence, Defense Evasion, Credential Access, Collection, and Command and Control, and a full remediation plan was documented (host isolation, C2 IP blocking, credential resets, forensic imaging, malware submission for reverse engineering, full reimage, detection signature updates, and stakeholder notification).

### 6. Proactive Threat Hunting Lab 2 (Large-Scale Multi-Attacker Campaign)

A 100,000-alert log was triaged and investigated using jq, starting with alert signature frequency, source IP, and destination port breakdowns. The investigation identified and attributed three distinct external attackers operating in a coordinated, multi-stage campaign spanning almost eight days:

- **203.0.113.42** — conducted Redis CONFIG command abuse against a Redis server, generating 30,000 alerts and achieving full remote control of the service.
- **185.22.1.5** (Italy, Momit SRL data center) — conducted MongoDB port scanning, a 20,000-attempt brute-force campaign timed for 4:40 AM to minimise the chance of detection, and ultimately confirmed exfiltration of the production MongoDB database (5,000 large data transfer alerts over roughly 8.3 hours).
- **95.161.220.10** (Moscow, Russia) — conducted NMAP reconnaissance scanning that fed directly into the subsequent brute-force phase.

Separately, internal analysis identified that over 250 internal workstations had already been compromised and were beaconing to an internal server disguised as social media traffic and vulnerability scan traffic, indicating this server was operating as an internal C2 relay well before the external database attacks began. A full attack timeline was constructed day by day, mapped comprehensively to MITRE ATT&CK (Execution, Persistence, Defense Evasion, Credential Access, Discovery, Lateral Movement, Command and Control, Exfiltration, and Impact), and the incident was formally justified as a critical security incident with a complete NIST-aligned incident response walkthrough covering Preparation, Detection and Analysis, Containment/Eradication/Recovery, and Post-Incident Activity.

## Key Findings Summary

- Both phishing samples were conclusively verdicted as malicious through domain inconsistencies, failed email authentication, and VirusTotal-confirmed malicious links and attachments.
- Six distinct network attack patterns were identified and validated purely from protocol-level evidence in Wireshark, without relying on signature-based detection.
- All three custom Suricata rules successfully detected their target behaviour in live testing, each grounded in a specific justification tied to real-world adversary tradecraft.
- Threat Hunting Lab 1 reconstructed a complete two-stage banking trojan infection chain (QakBot to Gozi) from raw pcap-derived alerts, fully mapped to MITRE ATT&CK with a remediation plan.
- Threat Hunting Lab 2 reconstructed an eight-day, three-attacker coordinated campaign against production databases from a 100,000-alert log, resulting in confirmed data exfiltration, and documented a complete NIST incident response lifecycle with lessons learned and remediation actions.

## MITRE ATT&CK Coverage

Techniques mapped across the project include, but are not limited to:

T1566.002 Spear Phishing, T1204 User Execution, T1059 Command and Scripting Interpreter, T1078 Valid Accounts, T1070.004 Indicator Removal, T1056.003 Input Capture, T1083 File and Directory Discovery, T1021 Remote Services, T1560.001 Archive Collected Data via Utility, T1071.001 Application Layer Protocol (Web Protocols), T1071.004 Application Layer Protocol (DNS), T1567.002 Exfiltration to Cloud Storage, T1036 Masquerading, T1027 Obfuscated Files or Information, T1055 Process Injection, T1562 Impair Defenses, T1555.003 Credentials from Web Browsers, T1056.004 Input Capture (Credential API Hooking), T1185 Man in the Browser, T1105 Ingress Tool Transfer, T1104 Multi-Stage Channels, T1008 Fallback Channels, T1573.002 Encrypted Channel (Asymmetric Cryptography), T1573 Encrypted Channel, T1098 Account Manipulation, T1110.001 Brute Force (Password Guessing), T1552 Unsecured Credentials, T1046 Network Service Scanning, T1595.001 Active Scanning, T1550 Use Alternate Authentication Material, T1090 Proxy, T1102 Web Service, T1048 Exfiltration Over Alternative Protocol, T1030 Data Transfer Size Limits, T1565 Data Manipulation, T1189 Drive-by Compromise, T1659 Web Content Injection.

## Report Structure

1. Introduction (Overview, Objectives, Scope, Analysis Environment, Tools Used, Disclaimer)
2. Executive Summary
3. Methodology
4. Mapping Adversary Behaviours Using MITRE ATT&CK
5. Phishing Email Forensics (Text Editor Analysis, Sandbox Analysis)
6. Network Packet Capture Analysis Lab 1
7. Network Packet Capture Analysis Lab 2
8. Intrusion Detection Systems (Rule Creation)
9. Proactive Threat Hunting Lab 1 (Setup, Hypothesis, Investigation, Attack Timeline, MITRE ATT&CK Mapping)
10. Proactive Threat Hunting Lab 2 (Setup, Analysis, Hypothesis, Detailed Investigation, Attack Overview, MITRE ATT&CK Mapping, Security Incident Justification)
11. Incident Response (NIST-aligned lifecycle for Lab 2)

The full report (PDF) is included in this repository and contains complete methodology write-ups, Wireshark and Suricata screenshots, jq commands with line-by-line explanations, VirusTotal and OSINT evidence, full attack timelines, and detailed remediation recommendations for every exercise.


