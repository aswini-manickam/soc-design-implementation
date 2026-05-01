[README (4).md](https://github.com/user-attachments/files/27271220/README.4.md)

# SOC Design & Implementation - Capstone Project

> Full Security Operations Centre infrastructure designed and deployed from scratch - covering network architecture, live threat detection with FortiGate NGFW, and real-time SIEM monitoring via Splunk.

[![SIEM](https://img.shields.io/badge/SIEM-Splunk-black?style=flat&logo=splunk)](https://www.splunk.com/)
[![Firewall](https://img.shields.io/badge/Firewall-FortiGate%20NGFW-EE3124?style=flat)](https://www.fortinet.com/)
[![Tool](https://img.shields.io/badge/Tool-Wireshark%20%7C%20MS%20Visio-blue?style=flat)]()
[![Environment](https://img.shields.io/badge/Environment-Controlled%20Lab-grey?style=flat)]()

---

## Project Overview

This capstone project covers the end-to-end **design and implementation of a SOC-integrated network infrastructure** - from initial architecture design through to live firewall deployment, SIEM configuration, and active penetration testing.

The project demonstrates a thorough understanding of **network security principles** and practical SOC operations: designing the network, building the detection stack, and validating it through structured penetration testing.

---

## Objectives

- Design a SOC-ready network infrastructure from the ground up
- Configure FortiGate NGFW with live threat detection capabilities
- Deploy Splunk as the SIEM platform with full log ingestion pipeline
- Validate the environment through web and network penetration testing
- Produce professional pentest reports from findings

---

## Tools & Technologies

| Tool / Platform | Role |
|----------------|------|
| **Splunk Enterprise** | SIEM - real-time log analysis and monitoring |
| **Splunk Universal Forwarder** | Log collection agents across environment nodes |
| **Splunk CIM** | Common Information Model - log normalisation |
| **FortiGate NGFW** | Next-Generation Firewall with IDS/IPS |
| **FortiGuard** | Threat intelligence feed for IDS/IPS signatures |
| **Wireshark** | Packet capture and deep traffic analysis |
| **MS Visio** | Network architecture and SOC design diagrams |

---

## Phase 1 - Network Architecture Design

### Design Scope

The network architecture was designed to support a fully integrated SOC capability, covering:

| Layer | Components |
|-------|------------|
| **Perimeter** | FortiGate NGFW, DMZ segment |
| **Core Network** | Routed LAN with subnetting scheme |
| **Server Zone** | Log sources, application servers |
| **SOC Zone** | Splunk SIEM, analyst workstations |
| **Management** | Out-of-band management network |

### Networking Concepts Applied

- **TCP/IP & OSI Model** - Protocol stack understanding applied to traffic flow design
- **DNS Architecture** - Internal and external DNS zone design
- **Subnetting** - CIDR-based IP addressing scheme across all network segments
- **Firewall Architecture** - Zone-based policy model, traffic segmentation, and least-privilege flow rules

### Deliverable

- Full network topology diagram (MS Visio) with annotated traffic flows
- IP addressing and subnetting plan
- Firewall zone and policy design document

---

##  Phase 2 - FortiGate NGFW Configuration

### Firewall Policies

- Defined zone-based firewall policies enforcing least-privilege traffic flows between segments
- Configured explicit deny rules with logging for all policy violations
- Applied NAT policies for outbound internet access and DMZ services

### IDS/IPS - FortiGuard Integration

| Feature | Configuration |
|---------|--------------|
| **Intrusion Prevention** | FortiGuard IPS signature profiles applied per policy |
| **Anomaly Detection** | Rate-based signatures for DoS and scanning activity |
| **SSL Inspection** | Deep packet inspection on outbound HTTPS traffic |
| **Application Control** | Application signatures applied to identify and control traffic types |
| **Web Filtering** | URL category-based filtering enforced on user traffic |

### Live Threat Detection

- FortiGate IPS events streamed to Splunk via syslog for correlation
- Tuned signature sensitivity to balance detection fidelity with alert noise

---

##  Phase 3 - Splunk SIEM Deployment

### Architecture

```
Log Sources (Servers, FortiGate, Workstations)
        │
        ▼
Splunk Universal Forwarders   ← Installed on each log source node
        │
        ▼
Splunk Indexer               ← Parses, indexes, and stores events
        │
        ▼
Splunk Search Head           ← Dashboards, alerts, and analyst queries
```

### Configuration

| Component | Implementation |
|-----------|---------------|
| **Universal Forwarders** | Deployed on all Windows servers, Linux nodes, and FortiGate syslog feed |
| **Splunk Add-ons** | Installed relevant TAs for Windows, FortiGate, and network data |
| **CIM (Common Information Model)** | Applied for log normalisation across diverse source types |
| **Dashboards** | Built operational dashboards for network traffic, authentication, and IPS events |
| **Alerts** | Configured threshold-based alerting for high-priority event categories |

---

##  Phase 4 - Penetration Testing & Validation

The deployed environment was validated through structured penetration testing.

### Web Application Testing

- Identified and tested OWASP Top 10 vulnerability classes in lab web applications
- SQL injection, authentication bypass, and session management testing
- Findings documented in structured pentest report format

### Network Penetration Testing

- Network reconnaissance and service enumeration across all segments
- Exploitation attempts to validate firewall policy enforcement
- Validated IDS/IPS detection coverage - confirmed alert generation for known attack signatures

### Reporting

- Produced structured penetration test reports covering methodology, findings, risk ratings, and remediation recommendations
- Reports formatted for both technical reviewers and non-technical stakeholders

---

## 📁 Repository Structure

```
soc-design-implementation/
│
├── README.md                            # This file
│
├── network-design/
│   ├── architecture-overview.md         # Network design narrative
│   ├── ip-addressing-plan.md            # Subnetting and IP scheme
│   ├── diagrams/                         # MS Visio exports (PNG/PDF)
│   └── firewall-zone-policy-design.md  # Zone model and policy intent
│
├── fortigate-ngfw/
│   ├── firewall-policy-summary.md       # Policy structure and rules
│   ├── ips-configuration.md             # IDS/IPS profile setup
│   └── ssl-inspection-setup.md          # SSL deep inspection config
│
├── splunk-siem/
│   ├── deployment-architecture.md       # Forwarder + indexer + SH layout
│   ├── add-ons-and-cim.md              # Installed add-ons and CIM mapping
│   ├── dashboards/                      # Dashboard screenshots
│   └── alert-rules.md                  # Configured alert definitions
│
├── penetration-testing/
│   ├── web-app-pentest/
│   │   ├── methodology.md
│   │   └── findings.md
│   └── network-pentest/
│       ├── methodology.md
│       └── findings.md
│
└── reports/
    └── pentest-report-template.md      # Report structure used
```

---

##  Key Outcomes

- Delivered a fully operational SOC-ready network infrastructure from design to deployment
- Achieved live threat detection via FortiGuard IPS with Splunk SIEM integration
- Validated detection coverage through structured penetration testing
- Produced professional pentest reports in industry-standard format

---

##  References

- [Splunk Documentation](https://docs.splunk.com/)
- [Splunk Common Information Model](https://docs.splunk.com/Documentation/CIM/)
- [FortiGate Administration Guide](https://docs.fortinet.com/product/fortigate/)
- [MITRE ATT&CK — Detection Engineering](https://attack.mitre.org/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

---

##  Disclaimer

All activities documented in this repository were conducted exclusively in **isolated, controlled lab environments** for cybersecurity training and professional development purposes. No configurations or tests were applied to any production systems or networks.

---

##  Author

**Aswini Manickam**  
[GitHub](https://github.com/aswinimanickam) · [LinkedIn](https://linkedin.com/in/aswinimanickam)

---

*Part of the [Cybersecurity Project Portfolio](https://github.com/aswinimanickam)*
