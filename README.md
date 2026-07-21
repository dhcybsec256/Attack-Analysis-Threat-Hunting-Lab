# Attack Analysis and Threat Hunting Project

Practical attack analysis and incident response project simulating real-world security operations across six exercises: adversary behaviour mapping (MITRE ATT&CK), phishing email forensics, network packet capture analysis, Intrusion Detection System (IDS) deployment with custom rule authoring, and two proactive threat hunting labs. All investigations were conducted inside isolated virtual machines using fictional, self-created scenarios.

> **Disclaimer:** This project was created solely for independent educational and portfolio purposes. All data, artifacts, incidents, IP addresses, and organisations described are fictional and not associated with any real persons, companies, or events.

---

## Overview

This project reflects the practical workflow of a SOC analyst or incident responder: mapping adversary tactics to structured frameworks, identifying malicious artifacts in raw data (emails, packet captures, logs), configuring and tuning detection tooling, and conducting evidence-based threat hunting to uncover and document full attack chains — all documented in accordance with incident response and threat intelligence best practices.

---

## Key Findings

- **MITRE ATT&CK Mapping** — Mapped a full 10-stage attack chain (spear phishing → credential theft → lateral movement → cloud exfiltration) to ATT&CK techniques for a simulated university phishing incident.
- **Phishing Forensics** — Analysed two phishing emails via manual header inspection and PhishTool sandbox, identifying spoofed domains, failed SPF/DKIM/DMARC checks, and malicious hyperlinks confirmed via VirusTotal, including one sample with a trojan-laced PDF attachment.
- **DDoS Traffic Analysis** — Identified and validated six distinct attack patterns in Wireshark (ICMP flood, TCP ACK flood/port scan, HTTP GET/POST floods, TCP SYN flood, CDN cache-bypass flood) using protocol-level evidence such as hardcoded sequence numbers, fixed window sizes, and identical User-Agents.
- **Custom Suricata IDS Rules** — Authored and tested four custom detection rules for C2-over-HTTP, HTTPS data exfiltration, and DNS tunnelling (long-domain and high-frequency variants), deployed and validated on Kali Linux.
- **Threat Hunting Lab 1** — Used jq to triage a Suricata alert log and uncover a QakBot → Gozi multi-stage banking trojan infection chain, including C2 beaconing, DNS manipulation, and credential theft, fully mapped to ATT&CK with an incident timeline and remediation plan.
- **Threat Hunting Lab 2** — Investigated a 100,000-alert log revealing an 8-day coordinated multi-attacker campaign (Redis exploitation, MongoDB brute-force, NMAP recon, internal C2 relay, and confirmed data exfiltration across 250+ compromised workstations), documented end-to-end under the NIST Incident Response framework.

---

## Skills Demonstrated

- MITRE ATT&CK technique mapping
- Email header forensics & phishing analysis (SPF/DKIM/DMARC)
- Network traffic analysis & DDoS attack identification (Wireshark)
- IDS deployment & custom Suricata rule writing (regex, thresholds, flow control)
- Log-based threat hunting with jq
- OSINT investigation (IP attribution, infrastructure analysis)
- Incident response documentation (NIST IR lifecycle)

---

## Tools Used

| Tool | Purpose |
|---|---|
| MITRE ATT&CK Framework | Adversary tactic, technique, and procedure (TTP) mapping |
| Wireshark | Packet capture analysis and protocol-level investigation |
| Suricata | IDS deployment, traffic monitoring, and custom rule detection |
| jq | JSON log parsing and command-line threat hunting |
| PhishTool | Phishing email sandbox and header analysis |
| VirusTotal | URL, file, and hash reputation analysis |
| whatismyipaddress.com | OSINT IP attribution and geolocation |
| Kali Linux | IDS host environment |

---

## Project Structure
├── docs/
│ └── Attack-Analysis-and-Threat-Hunting-Report.pdf
├── README.md
---

The full report (80 pages) contains the complete methodology, evidence, screenshots, custom Suricata rule justifications, jq query breakdowns, attack timelines, and MITRE ATT&CK mappings for all six exercises.
