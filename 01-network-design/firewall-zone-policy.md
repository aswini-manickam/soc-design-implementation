[firewall-zone-policy.md](https://github.com/user-attachments/files/27348550/firewall-zone-policy.md)
# Firewall Zone & Policy Design

## Zone Model

Three security zones enforced by FortiGate, each mapped to a physical interface. All inter-zone traffic is denied by default. Allow rules are explicit, minimal, and logged.

| Zone | Interface | Subnet | Trust Level |
|---|---|---|---|
| LINUX-GW | Port 1 | `10.200.1.0/24` | Untrusted (internet-facing) |
| Client | Port 2 | `192.168.20.0/28` | Client (semi-trusted) |
| SOC Operator | Port 3 | `10.0.1.0/24` | Trusted (SOC operator) |

---

## Firewall Policy Matrix

| # | Source Zone | Destination Zone | Service | Action | Logging | NAT |
|---|---|---|---|---|---|---|
| 1 | Client | LINUX-GW | HTTP, HTTPS, DNS | Allow | Yes | Yes |
| 2 | SOC Operator | LINUX-GW | HTTP, HTTPS, DNS | Allow | Yes | Yes |
| 3 | SOC Operator | Client | SMB (post-remediation: `10.0.1.10` only) | Allow | Yes | No |
| 4 | Client | SOC Operator | -| Deny | Yes | -|
| 5 | LINUX-GW | Client | -| Deny | Yes | -|
| 6 | LINUX-GW | SOC Operator | -| Deny | Yes | -|
| 99 | Any | Any | Any | Deny (implicit) | Yes | -|

> Policy 3 was tightened during penetration testing remediation. Pre-remediation, SOC Operator→Client allowed all hosts. Post-remediation, access was restricted to a named address object (`10.0.1.10` -SOC Operator Server) only, blocking the brute-force attack path from the pentest VM (`10.0.1.12`).

---

## IDS/IPS Profile -Applied Per Policy

FortiGuard IPS signature profiles applied to all inter-zone allow policies:

| Feature | Configuration |
|---|---|
| Intrusion Prevention | FortiGuard IPS signature profiles -all policies |
| Anomaly Detection | Rate-based signatures for DoS and port scanning |
| SSL Inspection | Deep packet inspection on outbound HTTPS (Policies 1, 2) |
| Application Control | Application signatures on user-facing policies |
| Web Filtering | URL category filtering on outbound HTTP/HTTPS |

---

## DoS Policy -Layer 4 Anomaly Protection

Separate IPv4 DoS policies created after penetration testing confirmed vulnerability to flood attacks.

### TCP SYN Flood -Client Inbound

| Parameter | Value |
|---|---|
| Incoming Interface | Port 2 (Client) |
| Source / Destination | All |
| L4 Anomaly | `tcp_syn_flood` -Block |
| Threshold | 200 packets/sec |
| Logging | Enabled |

**Validation:** Post-policy retest of SYN flood from `192.168.20.4` confirmed `clear_session` events in Splunk -`tcp_syn_flood, 56722 > threshold 200, repeats 2083674 since last log`. Client internet connectivity restored during active flood.

### ICMP Flood (Smurf) -Client Inbound

| Parameter | Value |
|---|---|
| Incoming Interface | Port 2 (Client) |
| Source / Destination | All |
| L4 Anomaly | `icmp_flood`, `icmp_src_session`, `icmp_dst_session` -Block |
| Threshold | 200 packets/sec |
| Logging | Enabled |

**Validation:** DC Server CPU dropped from 100% utilization to near-baseline within seconds of policy activation. Splunk confirmed new `icmp_flood clear_session` events from DoS policy.

---

## NAT Configuration

| Rule | Source | Translated Address | Purpose |
|---|---|---|---|
| Outbound NAT | `192.168.20.0/28` | FortiGate Port 1 IP | Client internet access |
| Outbound NAT | `10.0.1.0/24` | FortiGate Port 1 IP | SOC Operator internet access |

No inbound NAT configured -no services published from internal segments to the internet during this engagement.

---

## Evidence

| Screenshot | Location | What it shows |
|---|---|---|
| FortiGate policy list (pre-remediation) | [`../evidence/pentest/fortigate-policy-pre-remediation.png`](../evidence/pentest/) | Broad SOC Operator→Client allow rule before tightening |
| FortiGate policy list (post-remediation) | [`../evidence/pentest/fortigate-policy-post-remediation.png`](../evidence/pentest/) | Address object `10.0.1.10` applied, all other SOC Operator hosts blocked |
| DoS policy -TCP SYN flood config | [`../evidence/pentest/fortigate-dos-policy-synflood.png`](../evidence/pentest/) | L4 anomaly config with block action and 200pps threshold |
| DoS policy -ICMP flood config | [`../evidence/pentest/fortigate-dos-policy-icmp.png`](../evidence/pentest/) | ICMP flood, src/dst session anomalies enabled |
