[README.md](https://github.com/user-attachments/files/27310726/README.md)
# SOC Design & Implementation

> End-to-end design and deployment of a Security Operations Centre for a simulated enterprise client - covering network architecture, FortiGate NGFW configuration, Splunk SIEM integration, and adversarial validation across three network segments. All detection coverage was confirmed through structured penetration testing.

[![SIEM](https://img.shields.io/badge/SIEM-Splunk_Enterprise-black?style=flat&logo=splunk)](https://www.splunk.com/)
[![Firewall](https://img.shields.io/badge/Firewall-FortiGate_NGFW-EE3124?style=flat)](https://www.fortinet.com/)
[![Recon](https://img.shields.io/badge/Recon-Nessus_%7C_Nmap-blue?style=flat)]()
[![Exploit](https://img.shields.io/badge/Exploit-Metasploit_%7C_Impacket-red?style=flat)]()
[![Analysis](https://img.shields.io/badge/Analysis-Wireshark_%7C_Burp_Suite-grey?style=flat)]()
[![Environment](https://img.shields.io/badge/Environment-Controlled_Lab-lightgrey?style=flat)]()

---

## Environment Overview

Three network segments separated by a FortiGate NGFW, with Splunk Enterprise deployed as the centralized SIEM. All penetration testing was conducted from within the lab boundary against deliberately misconfigured hosts, designed to validate detection coverage.

| Segment | Subnet | Role |
|---|---|---|
| client | `192.168.20.0/28` | Domain-joined Windows servers, web application host |
| SOC operator | `10.0.1.0/24` | Splunk indexer, FortiGate management, analyst workstations |
| Linux (gateway) | `10.200.1.0/24` | Internet-facing gateway, web server |

→ Network topology diagram: [`./diagrams/network-topology-annotated.png`](./diagrams/network-topology-annotated.png)

---

## Project Phases

### Phase 1 - Network Architecture & Firewall Design

Designed a zone-based network model enforcing least-privilege traffic flows between the  client network, SOC network, and the Linux internet gateway. FortiGate interfaces were assigned per segment (Port 1 → Linux, Port 2 → client, Port 3 → SOC), with NAT policies controlling outbound access and explicit deny rules logging all policy violations.

**Key outcome:** Zone isolation verified - SOC hosts could not reach Client internal resources without explicit firewall policy allow rules. Cross-segment access was enforced to a single trusted server (`10.0.1.10`) following remediation of the SMB exploit.

→ [`./01-network-design/architecture-overview.md`](./01-network-design/architecture-overview.md)  
→ [`./01-network-design/ip-subnetting-plan.md`](./01-network-design/ip-subnetting-plan.md)  
→ [`./01-network-design/firewall-zone-policy.md`](./01-network-design/firewall-zone-policy.md)

---

### Phase 2 - FortiGate NGFW Configuration

Configured FortiGuard IPS signature profiles per zone-based policy, with rate-based anomaly detection for DoS and scanning activity, SSL deep packet inspection on outbound HTTPS traffic, and application control signatures applied to user traffic.

DoS protection was validated empirically: the initial threshold of 200 packets/sec for TCP SYN flood detection was confirmed effective during testing, blocking the flood within seconds of attack initiation while leaving legitimate traffic unaffected.

**Key outcome:** IPv4 DoS Policy blocked a 3-million-event SYN flood within the suppression window. Splunk logged 8 `clear_session` events confirming policy enforcement - `tcp_syn_flood, 56722 > threshold 200, repeats 2083674 since last log`.

→ [`./02-fortigate-ngfw/policy-summary.md`](./02-fortigate-ngfw/policy-summary.md)  
→ [`./02-fortigate-ngfw/ips-profile-config.md`](./02-fortigate-ngfw/ips-profile-config.md)  
→ [`./02-fortigate-ngfw/dos-policy-config.md`](./02-fortigate-ngfw/dos-policy-config.md)

---

### Phase 3 - Splunk SIEM Deployment

Deployed Splunk Enterprise on the SOC Windows Server with Universal Forwarders installed on all Windows domain nodes, Linux hosts, and FortiGate syslog feed (UDP 514). CIM normalization applied across all source types using relevant Splunk Technology Add-ons.

```
Log Sources (Client DC, SOC Server, FortiGate, Linux Gateway)
        │
        ▼  [Universal Forwarders - TCP 9997]
        │
        ▼
Splunk Indexer (SOC Server)   ← Parsing, CIM normalization, indexing
        │
        ▼
Splunk Search Head                ← Dashboards, threshold alerts, SPL queries
```

Operational dashboards built for: network traffic volume by segment, FortiGate IPS events by signature category, authentication failure spikes, and DoS rate monitoring.

**Key outcome:** End-to-end log pipeline confirmed - FortiGate syslog events visible in Splunk Data Summary under `sourcetype=fortigate_log` within seconds of generation. IPS alerts correlated with attack timelines during all penetration tests.

→ [`./03-splunk-siem/deployment-architecture.md`](./03-splunk-siem/deployment-architecture.md)  
→ [`./03-splunk-siem/alert-rules.md`](./03-splunk-siem/alert-rules.md)  
→ [`./queries/splunk-detection-queries.md`](./queries/splunk-detection-queries.md)  
→ [`./evidence/splunk-dashboards/`](./evidence/splunk-dashboards/)

---

### Phase 4 - Penetration Testing & Detection Validation

Structured penetration testing across all three network segments, combining Nessus vulnerability scanning with targeted exploitation. Each attack was logged, correlated in Splunk, and followed by a security control implementation and retest cycle.

---

## Findings Summary

| Finding | Segment | Risk | MITRE Technique | Detection Validated | Status |
|---|---|---|---|---|---|
| SMB brute force + Meterpreter shell | client | **Critical** | T1110.001, T1059 | Splunk auth failure spike | Remediated - firewall policy + password policy |
| TCP SYN flood (DoS) | client | **High** | T1498.001 | FortiGate DoS Policy + Splunk | Remediated - IPv4 DoS Policy enforced |
| SQL injection (Juice Shop) | Linux / Web | **High** | T1190 | Web server logs | Recommended - input validation |
| Web app brute force (Burp Suite) | Linux / Web | **High** | T1110 | Web server logs | Recommended - password complexity + rate limiting |
| Smurf attack (ICMP flood) | client | **High** | T1498.001 | Splunk - 73% of FortiGate logs from spoofed src | Remediated - ICMP DoS Policy |
| Ransomware simulation (CashCat) | client | **High** | T1486, T1059 | Meterpreter session logs | Mitigated - privilege controls reviewed |
| IDOR (Juice Shop basket access) | Linux / Web | **Medium** | T1078 | Web server access logs | Recommended - session token scope, ID hashing |
| ARP poisoning / MitM | client | **Medium** | T1557.002 | Wireshark / traffic analysis | Recommended - dynamic ARP inspection |

→ Full findings detail: [`./04-pentest/`](./04-pentest/)  
→ MITRE ATT&CK mapping: [`./04-pentest/mitre-attack-mapping.md`](./04-pentest/mitre-attack-mapping.md)

---

## Detection Engineering

### Splunk SPL - Detection Queries

**DoS source identification (SYN flood)**
```spl
index=fortigate sourcetype=fortigate_log subtype=ips
| stats count by src_ip, attack_name
| where count > 100
| sort -count
```
Identifies source IPs generating more than 100 IPS events within the search window. Used during SYN flood testing to confirm `192.168.20.4` as the attack source and verify policy enforcement.

---

**Authentication failure spike - brute force detection**
```spl
index=windows sourcetype=WinEventLog:Security EventCode=4625
| bucket _time span=5m
| stats count by _time, src_ip, user
| where count > 20
| sort -count
```
Detects accounts receiving more than 20 failed authentication attempts within a 5-minute window. During SMB brute force testing, this query surfaced the attack against the Administrator account within seconds of the Metasploit module completing.

---

**FortiGate DoS policy enforcement - clear session events**
```spl
index=fortigate sourcetype=fortigate_log action="clear_session"
| stats count by src_ip, policyname, attack_name
| sort -count
```
Confirms DoS policy is actively blocking traffic. The `tcp_syn_flood` and `icmp_flood` entries appeared here during retest cycles, confirming both policies were enforced correctly.

---

**IPS event volume by signature - tuning baseline**
```spl
index=fortigate sourcetype=fortigate_log subtype=ips
| timechart span=1m count by attack_name
```
Used to establish a detection baseline and identify noisy signatures requiring threshold adjustment. Provides a per-minute event rate per signature to distinguish genuine attack traffic from legitimate bursts.

---

**IDOR and unauthorized access - web application log triage**
```spl
index=webserver sourcetype=access_combined
| rex field=_raw "GET /rest/basket/(?<basket_id>\d+)"
| stats count by src_ip, basket_id, status
| where status=200 AND count > 3
```
Surfaces repeated successful basket ID enumeration from a single source - the pattern consistent with the IDOR exploit observed against the Juice Shop application.

---

→ All queries as separate files: [`./queries/`](./queries/)

---

## Key Evidence

| Evidence Item | Location | What it shows |
|---|---|---|
| Nessus scan - client (97 vulns found) | [`./evidence/nessus/nessus-client-scan-summary.png`](./evidence/nessus/) | Reconnaissance phase: critical and high vulns used to select pentest targets |
| Nessus scan - SOC network (SSL misconfig, cleartext Telnet) | [`./evidence/nessus/nessus-soc-findings.png`](./evidence/nessus/) | FortiGate management surface exposed via Telnet - medium risk |
| Splunk - SYN flood detection (~3M events in 2-3 min) | [`./evidence/splunk-dashboards/splunk-synflood-detection.png`](./evidence/splunk-dashboards/) | FortiGate IPS event volume confirming DoS detection |
| Splunk - DoS policy clear_session events | [`./evidence/splunk-dashboards/splunk-dos-policy-enforcement.png`](./evidence/splunk-dashboards/) | Post-remediation retest: 8 events confirming policy blocked flood |
| Splunk - SMB brute force auth failures | [`./evidence/splunk-dashboards/splunk-smb-auth-failures.png`](./evidence/splunk-dashboards/) | High-volume 4625 events surfaced during Metasploit brute force run |
| Meterpreter session established (DCSrv1) | [`./evidence/pentest/metasploit-smb-meterpreter-session.png`](./evidence/pentest/) | Critical: full domain controller compromise via SMB brute force + HTA reverse shell |
| FortiGate policy - cross-segment access restricted | [`./evidence/pentest/fortigate-policy-post-remediation.png`](./evidence/pentest/) | Remediation confirmed: SOC→client blocked except `10.0.1.10` |
| IDOR - basket ID tamper via ZAP | [`./evidence/pentest/idor-zap-basket-tamper.png`](./evidence/pentest/) | Unauthenticated access to another user's basket via bearer token misuse |
| Smurf attack - DC processor at 100% | [`./evidence/pentest/smurf-dc-processor-utilization.png`](./evidence/pentest/) | DoS impact confirmed; resolved after ICMP DoS policy applied |
| Splunk - Smurf attack log pattern (73% ICMP from spoofed src) | [`./evidence/splunk-dashboards/splunk-smurf-patterns.png`](./evidence/splunk-dashboards/) | Pattern analysis tab showing ICMP flood dominance in FortiGate logs |

---

## Repository Structure

```
soc-design-implementation/
│
├── README.md
│
├── 01-network-design/
│   ├── architecture-overview.md
│   ├── ip-subnetting-plan.md
│   └── firewall-zone-policy.md
│
├── 02-fortigate-ngfw/
│   ├── policy-summary.md
│   ├── ips-profile-config.md
│   └── dos-policy-config.md
│
├── 03-splunk-siem/
│   ├── deployment-architecture.md
│   └── alert-rules.md
│
├── 04-pentest/
│   ├── mitre-attack-mapping.md
│   ├── web-app/
│   │   └── findings.md
│   └── network/
│       └── findings.md
│
├── queries/
│   ├── splunk-detection-queries.md
│   ├── query-01-dos-source-identification.spl
│   ├── query-02-auth-failure-spike.spl
│   ├── query-03-dos-policy-enforcement.spl
│   ├── query-04-ips-volume-tuning-baseline.spl
│   └── query-05-idor-web-access-triage.spl
│
├── diagrams/
│   ├── network-topology-annotated.png
│   ├── splunk-deployment-architecture.png
│   └── mitre-attack-chain.png
│
├── evidence/
│   ├── nessus/
│   │   ├── nessus-client-scan-summary.png
│   │   ├── nessus-soc-findings.png
│   │   └── nessus-linux-scan-summary.png
│   ├── splunk-dashboards/
│   │   ├── splunk-synflood-detection.png
│   │   ├── splunk-dos-policy-enforcement.png
│   │   ├── splunk-smb-auth-failures.png
│   │   └── splunk-smurf-patterns.png
│   ├── pentest/
│   │   ├── metasploit-smb-meterpreter-session.png
│   │   ├── fortigate-policy-post-remediation.png
│   │   ├── idor-zap-basket-tamper.png
│   │   └── smurf-dc-processor-utilization.png
│   └── sample-logs/
│       ├── fortigate-synflood-syslog-sanitized.txt
│       └── splunk-smb-auth-failure-events-sanitized.txt
│
└── reports/
    ├── SOC-design-report-redacted.pdf
    └── penetration-test-report-redacted.pdf
```

---

## MITRE ATT&CK Coverage

| Tactic | Technique | ID | Attack Executed |
|---|---|---|---|
| Reconnaissance | Active Scanning | T1595 | Nmap port scan across all segments |
| Initial Access | Exploit Public-Facing Application | T1190 | SQL injection (Juice Shop admin bypass) |
| Credential Access | Brute Force: Password Spraying | T1110.001 | SMB brute force against Administrator account |
| Credential Access | Brute Force: Password Guessing | T1110 | Burp Suite intruder against Juice Shop login |
| Execution | Command and Scripting Interpreter | T1059 | Meterpreter post-exploitation, mshta.exe HTA execution |
| Command & Control | Non-Standard Port | T1571 | Reverse TCP via Metasploit on port 4444 |
| Collection | Adversary-in-the-Middle | T1557.002 | ARP poisoning / MitM on client segment |
| Impact | Network Denial of Service | T1498.001 | TCP SYN flood, Smurf ICMP flood |
| Impact | Data Encrypted for Impact | T1486 | CashCat ransomware simulator on DCSrv1 |
| Discovery | Account Discovery | T1087 | Nessus credentialed scan - SMB user enumeration |

→ Full mapping with detection notes: [`./04-pentest/mitre-attack-mapping.md`](./04-pentest/mitre-attack-mapping.md)

---

## Reports

| Report | Description |
|---|---|
| [`SOC-design-report-redacted.pdf`](./reports/SOC-design-report-redacted.pdf) | SOC architecture, SIEM deployment, and firewall configuration documentation |
| [`penetration-test-report-redacted.pdf`](./reports/penetration-test-report-redacted.pdf) | Full pentest findings with reproduction steps, logs, risk ratings, and remediation |

---

## References

- [Splunk Documentation](https://docs.splunk.com/)
- [Splunk CIM (Common Information Model)](https://docs.splunk.com/Documentation/CIM/)
- [FortiGate Administration Guide](https://docs.fortinet.com/product/fortigate/)
- [MITRE ATT&CK Framework](https://attack.mitre.org/)
- [OWASP Web Security Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)
- [Tenable Nessus Documentation](https://docs.tenable.com/nessus/)

---

## Disclaimer

All activities documented in this repository were conducted exclusively in isolated, controlled lab environments for cybersecurity training and professional development. No configurations or tests were applied to production systems or networks.

---

*[Aswini Manickam](https://github.com/aswini-manickam) · [LinkedIn](https://linkedin.com/in/aswini-manickam)*
