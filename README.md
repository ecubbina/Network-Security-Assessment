# Network Security Assessment – Metasploitable 2

![Kali Linux](https://img.shields.io/badge/Platform-Kali%20Linux-blue)
![Nmap](https://img.shields.io/badge/Tool-Nmap-informational)
![Wireshark](https://img.shields.io/badge/Tool-Wireshark-informational)
![Security Assessment](https://img.shields.io/badge/Project-Network%20Security%20Assessment-red)
![Lab](https://img.shields.io/badge/Environment-Isolated%20Lab-success)

## 📌 Project Overview

This project documents an authorized network security assessment performed against an intentionally vulnerable **Metasploitable 2** virtual machine in an isolated VMware laboratory environment.

The assessment focused on identifying exposed network services, insecure configurations, authentication weaknesses, and potential security risks. Selected findings were manually validated and supported with network traffic analysis using Wireshark.

The project follows a simplified professional security assessment workflow:

**Discovery → Enumeration → Validation → Traffic Analysis → Risk Assessment → Remediation → Reporting**

---

## 🎯 Objectives

The primary objectives of this assessment were to:

- Identify accessible network services and open ports.
- Enumerate service versions and configurations.
- Identify potentially insecure services and authentication mechanisms.
- Manually validate selected security findings.
- Analyze network traffic associated with identified services.
- Assess security impact and risk.
- Develop practical remediation recommendations.
- Produce professional security assessment documentation.

---

## 🧪 Lab Environment

The assessment was conducted entirely within an isolated VMware Host-Only network.

| System | Role | IP Address |
|---|---|---|
| Kali Linux | Assessment / Security Testing Host | `192.168.27.128` |
| Metasploitable 2 | Intentionally Vulnerable Target | `192.168.27.129` |
| VMware Network | Isolated Host-Only Network | `192.168.27.0/24` |

### Lab Architecture

```text
                    VMware VMnet1
                 192.168.27.0/24
                        │
            ┌───────────┴───────────┐
            │                       │
            ▼                       ▼
       Kali Linux             Metasploitable 2
    192.168.27.128             192.168.27.129
     Assessment Host               Target
            │                       │
            └─────── Network ───────┘
                    Analysis
