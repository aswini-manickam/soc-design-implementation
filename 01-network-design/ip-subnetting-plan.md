[ip-subnetting-plan.md](https://github.com/user-attachments/files/27349107/ip-subnetting-plan.md)
# IP Addressing & Subnetting Plan

## Addressing Scheme

| Segment | Network Address | Subnet Mask | CIDR | Usable Range | Broadcast |
|---|---|---|---|---|---|
| Linux (gateway) | `10.200.1.0` | `255.255.255.0` | `/24` | `10.200.1.1 – 10.200.1.254` | `10.200.1.255` |
| Client | `192.168.20.0` | `255.255.255.240` | `/28` | `192.168.20.1 – 192.168.20.14` | `192.168.20.15` |
| SOC Operator | `10.0.1.0` | `255.255.255.0` | `/24` | `10.0.1.1 – 10.0.1.254` | `10.0.1.255` |

---

## Host Assignments

### Client - `192.168.20.0/28`

| Host | IP Address | Role |
|---|---|---|
| FortiGate Port 2 | `192.168.20.2` | Default gateway |
| DCSrv1 | `192.168.20.10` | Domain Controller (Windows Server) |
| Kali (pentest VM) | `192.168.20.4` | Penetration testing host |
| Broadcast | `192.168.20.15` | - |

> `/28` chosen deliberately - 14 usable hosts is appropriate for a small client LAN and demonstrates subnetting efficiency over a wasteful `/24`.

### SOC Operator - `10.0.1.0/24`

| Host | IP Address | Role |
|---|---|---|
| FortiGate Port 3 | `10.0.1.254` | Default gateway |
| SOC Operator Server | `10.0.1.10` | Splunk indexer, AD domain controller |
| Pentest VM (Kali) | `10.0.1.12` | Attack source for cross-segment tests |

### Linux Gateway - `10.200.1.0/24`

| Host | IP Address | Role |
|---|---|---|
| FortiGate Port 1 | `10.200.1.254` | Default gateway |
| Web server (Juice Shop) | `10.200.1.150` | Web application pentest target |
| FortiManager | `10.200.1.x` | FortiGate centralized management |

---

## Subnetting Rationale

- **Client `/28`** - tightly scoped client segment; limits blast radius of any lateral movement to 14 hosts maximum
- **SOC Operator `/24`** - wider SOC operator range to accommodate future analyst workstation growth without renumbering
- **Linux `/24`** - internet gateway segment; broad range to support multiple DMZ-style services if required

---

## DNS Architecture

- Client domain nodes resolve internally via DCSrv1 Active Directory DNS (`192.168.20.10`)
- SOC Operator nodes resolve via SOC Operator AD DNS server (`10.0.1.10`)
- Both segments forward external DNS queries outbound via FortiGate Port 1 NAT

---

## Relevance to Penetration Testing

The `/28` on Client constrained the Nessus scan target range - only 14 addresses to enumerate, making host discovery faster and the resulting findings more precise. The SMB brute force attack originated from `10.0.1.12` (SOC Operator) targeting `192.168.20.10` (Client DCSrv1), crossing the inter-zone boundary - which validated that the firewall policy was the only control preventing that path prior to remediation.
