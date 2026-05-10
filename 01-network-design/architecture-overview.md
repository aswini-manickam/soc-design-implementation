[architecture-overview (1).md](https://github.com/user-attachments/files/27347817/architecture-overview.1.md)
# Network Architecture Overview

## Environment Summary

A three-segment enterprise network separated by a FortiGate NGFW, with Splunk Enterprise deployed on the SOC segment as the centralized SIEM. The architecture simulates a managed security service provider (SOC operator) operating a SOC on behalf of a client (Client), with a Linux host acting as the internet-facing gateway.

---

## Network Segments

| Segment | Subnet | Role | Key Hosts |
|---|---|---|---|
| Client | `192.168.20.0/28` | Client domain network | DCSrv1 (Domain Controller), web app host |
| SOC operator | `10.0.1.0/24` | SOC platform, analyst workstations | Splunk indexer, FortiGate management |
| Linux (gateway) | `10.200.1.0/24` | Internet-facing gateway | Web server (Juice Shop), FortiManager |

---

## FortiGate Interface Assignment

| FortiGate Port | Connected Segment | IP Address |
|---|---|---|
| Port 1 | Linux (internet gateway) | `10.200.1.254` |
| Port 2 | Client LAN | `192.168.20.2` |
| Port 3 | SOC operator) LAN | `10.0.1.254` |

---

## Zone Model

Traffic between all three segments is controlled by zone-based firewall policies enforcing least-privilege flows. No segment has unrestricted access to another. Cross-segment communication requires an explicit allow rule.

```
[Internet]
    │
    ▼
[Linux Gateway -10.200.1.0/24]
    │
    ▼
[FortiGate NGFW]
    ├──► Client -192.168.20.0/28  (Port 2)
    └──► SOC operator) SOC -10.0.1.0/24     (Port 3)
```

---

## Design Principles Applied

- **Least privilege** -all inter-zone traffic denied by default; allow rules created only for required services
- **Explicit deny with logging** -all policy violations logged to FortiGate syslog, forwarded to Splunk
- **NAT at perimeter** -outbound internet access via NAT on Port 1 only; no direct external access to internal segments
- **Out-of-band management** -FortiGate management interface accessible only from SOC operator SOC segment
- **Log source coverage** -Universal Forwarders deployed on all Windows domain nodes, Linux hosts, and FortiGate syslog stream to ensure full SIEM visibility

---

## Diagram

→ [`../diagrams/network-topology-annotated.png`](https://github.com/aswini-manickam/soc-design-implementation/blob/main/diagrams/network-topology-annotated.png)

*Export your Visio file as PNG and place it at the path above. Annotate each segment with its subnet label and the FortiGate port connecting it.*

---

## Key Outcome

Zone isolation verified through penetration testing: cross-segment attack from SOC operator) (`10.0.1.12`) to Client (`192.168.20.0/28`) was blocked after firewall policy remediation. Only the explicitly allowed SOC operator) server (`10.0.1.10`) retained access -confirmed by failed SMB brute force retest from all other SOC operator hosts.
