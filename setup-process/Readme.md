# Purple Team Home Lab (pfSense + AD + Wazuh + Kali) Setup Process

This readme documents a realistic **purple-team home lab** built in **VirtualBox**:
- **pfSense** as the firewall/router (WAN/LAB-LAN/MGMT/ATTACK)
- **Windows Server (DC01)** running **AD DS + DNS**
- **Windows client (WIN11-CLIENT01)** joined to the domain
- **Wazuh (Ubuntu)** as SIEM/XDR (manager + dashboard)
- **Kali Linux** on a dedicated ATTACK segment

The goal is to support a repeatable loop:
**Attack → Detect → Harden → Attack**, with clean documentation and screenshots.

---

## Table of Contents
- [1. Topology & IP Plan](#1-topology--ip-plan)
- [2. VirtualBox Networking](#2-virtualbox-networking)
- [3. pfSense Installation & Config](#3-pfsense-installation--base-config)
- [4. Host Access to pfSense](#4-host-access-to-pfsense-mgmt-network)
- [5. DC01 (AD DS + DNS)](#5-dc01-ad-ds--dns)
- [6. WIN11-CLIENT01 Networking & Domain](#6-win11-client01-networking--domain-join)
- [7. Group Policy Auditing Baseline](#7-group-policy-auditing-baseline)
- [8. Sysmon Installation](#8-sysmon-installation)
- [9. Wazuh Server (Ubuntu) + Agents](#9-wazuh-server-ubuntu--agents)
- [10. Sysmon → Wazuh Ingestion](#10-sysmon--wazuh-ingestion)
- [11. ATTACK Network + Kali](#11-attack-network--kali)
- [12. VirtualBox Overview](#12-virtualbox-overview)
- [Troubleshooting Highlights](#troubleshooting-highlights)
- [Next Steps: Purple Team Scenarios](#next-steps-purple-team-scenarios)

---

## 1. Topology & IP Plan

**Networks**
- **LAB-LAN:** `10.10.10.0/24` (domain + endpoints)
- **MGMT:** `10.10.20.0/24` (host → pfSense management UI)
- **ATTACK:** `10.10.30.0/24` (Kali segment)

**Key IPs**
| Component | Interface/Network | IP |
|---|---|---|
| pfSense | LAN (LAB-LAN) | `10.10.10.1` |
| pfSense | OPT1 (MGMT) | `10.10.20.1` |
| pfSense | OPT2 (ATTACK) | `10.10.30.1` |
| DC01 | LAB-LAN | `10.10.10.10` |
| WIN11-CLIENT01 | LAB-LAN (DHCP) | `10.10.10.x` |
| WAZUH01 | LAB-LAN (DHCP/Static) | `10.10.10.x` |
| KALI01 | ATTACK (DHCP) | `10.10.30.100` |

📸 **Screenshot placeholder:** *Topology overview / IP plan*  
- `screenshots/setup/??-topology.png` *i will put general machine overview image here*

---

## 2. VirtualBox Networking

I used VirtualBox internal networks to isolate and control traffic:

- **Internal Network**: `LAB-LAN`
- **Internal Network**: `ATTACK`
- **Host-only**: `VirtualBox Host-Only Adapter`

📸 **Screenshot placeholder:** *Overall VirtualBox machine setup / adapters*  
- `screenshots/setup/33-overall-machine-setup-virtualbox.png`

---

## 3. pfSense Installation & Base Config

### 3.1 Install pfSense
- Download pfSense image (Netgate installer).
- Boot VM from ISO and install pfSense.
- Assign interfaces (WAN + LAN initially).

📸 **Screenshot placeholders:**
- `screenshots/machines/pfsense/01-pfsense-setup.png`
- `screenshots/machines/pfsense/02-pfsense-console.png`

### 3.2 Configure pfSense Interfaces
After install, i configured:
- **WAN:** DHCP (VirtualBox NAT)
- **LAN:** Static `10.10.10.1/24`
- **MGMT (OPT1):** Static `10.10.20.1/24`
- **ATTACK (OPT2):** Static `10.10.30.1/24` *(i added this later)*

📸 **Screenshot placeholders:**
- `screenshots/machines/pfsense/03-pfsense-ip-configs.png`

---

## 4. Host Access to pfSense (MGMT network)

### 4.1 Enable ICMP on Windows host (for ping tests)
At one point, ping failed due to Windows host firewall rules blocking ICMP.  
I enabled ICMP inbound rules so connectivity tests became reliable.

📸 **Screenshot placeholders:**
- `screenshots/problems/04-host-firewall-enable-icmp.png`

### 4.2 Log into pfSense Web UI
- From the host, i accessed pfSense via:
  - `https://10.10.20.1`

📸 **Screenshot placeholders:**
- `screenshots/machines/pfsense/05-pfsense-login-host.png`
- `screenshots/machines/pfsense/06-logged-in-pfsense-host.png`

### 4.3 MGMT Firewall Rules (WebUI access)
Created/adjusted firewall rules to allow MGMT subnet access to pfSense WebConfigurator.

📸 **Screenshot placeholders:**
- `screenshots/machines/pfsense/07-rules-firewall-host-pfsense.png`

---

## 5. DC01 (AD DS + DNS)

### 5.1 Set Static IP on DC01
DC01 was configured on LAB-LAN:
- IP: `10.10.10.10/24`
- Gateway: `10.10.10.1`
- DNS: **itself** (`10.10.10.10` or later `127.0.0.1` depending on DNS strategy)

📸 **Screenshot placeholders:**
- `screenshots/machines/dc01/08-dc01-initial-ip.png`

### 5.2 Install AD DS + DNS
- Installed **Active Directory Domain Services**
- Promoted server to Domain Controller
- Created domain: `vilius.lab`

📸 **Screenshot placeholders:**
- `screenshots/machines/dc01/09-dc01-ad-ds-install.png`
- `screenshots/machines/dc01/10-dc01-ad-ds-success.png`

### 5.3 OU Structure
Created OUs for clean policy targeting:
- `Admins`
- `Users_Org`
- `Servers`
- `Workstations`
- `ServiceAccounts`

Note: Built-in `Users` container is **not** an OU and should be left as it is.

---

## 6. WIN11-CLIENT01 Networking & Domain Join

### 6.1 Fix “169.254.x.x” (no DHCP) issue
The client originally got an APIPA address, meaning it wasn’t receiving DHCP.
Fix: Ensure VirtualBox NIC is connected to **Internal Network: LAB-LAN** and pfSense LAN DHCP is enabled.

📸 **Screenshot placeholders:**
- `screenshots/machines/win11/11-client01-network-setup.png`

### 6.2 DNS Troubleshooting (Client)
I encountered DNS timeouts during `nslookup` and validated:
- Ping to `10.10.10.1` (pfSense) and `10.10.10.10` (DC01)
- DNS resolution issues were fixed through a combination of firewall/routing/DNS-forwarding adjustments.

📸 **Screenshot placeholders:**
- `screenshots/problems/12-client01-ipconfig-dns-problem.png`
- `screenshots/problems/20-dc01-nslookup-dns-times-out.png`
- `screenshots/problems/21-client01-nslookup-works.png`

### 6.3 Domain Join
Important discovery: **Windows 11 Home cannot join a domain**, so I upgraded to **Windows 11 Pro** using generic key.

Then joined:
- Domain: `vilius.lab`
- Credentials: `vilius\ADMIN01`

📸 **Screenshot placeholders:**
- `screenshots/machines/win11/13-client01-domain-change.png`

### 6.4 Move Computer to Workstations OU
Moved `WIN11-CLIENT01` computer object into `Workstations` OU for correct GPO targeting.

📸 **Screenshot placeholders:**
- `screenshots/machines/dc01/14-dc01-client01-workstations.png`

### 6.5 Validate Domain Trust
Ran domain discovery test:
- `nltest /dsgetdc:vilius.lab`

📸 **Screenshot placeholders:**
- `screenshots/machines/win11/15-client01-domain-test-nltest.png`

---

## 7. Group Policy Auditing Baseline

Created a domain-linked GPO to enable **Advanced Audit Policy**:
- Credential validation (success/failure)
- Logon/logoff (success/failure)
- Special logon
- Policy change
- System integrity

Validated logs:
- On DC01: `4624` and `4672` appear for admin logons
- On WIN11: `4624` appears for standard logons (no `4672` expected for normal users)

📸 **Screenshot placeholders:**
- `screenshots/machines/dc01/16-dc01-new-gpo.png`
- `screenshots/machines/dc01/17-dc01-gpo-audit-configuration.png`
- `screenshots/machines/dc01/18-dc01-audit-events.png`

---

## 8. Sysmon Installation

Installed Sysmon with a baseline config (SwiftOnSecurity style config is a common choice):
- Verified Sysmon service running
- Verified Sysmon Operational logs exist

📸 **Screenshot placeholders:**
- `screenshots/machines/dc01/22-dc01-sysmon-download.png`
- `screenshots/machines/dc01/23-dc01-sysmon-check.png`

> Note: Sysmon Event ID 3 (NetworkConnect) may not always appear due to config filtering. That’s normal.

---

## 9. Wazuh Server (Ubuntu) + Agents

### 9.1 Install Wazuh Server
Installed Wazuh “all-in-one” (manager + indexer + dashboard).  
Some installer checks may require `--ignore-check` depending on the Ubuntu version.

📸 **Screenshot placeholders:**
- `screenshots/machines/wazuh/24-client01-wazuh-dashboard.png`

### 9.2 Install Wazuh Agents on Windows machines
Installed agents on:
- **DC01**
- **WIN11-CLIENT01**

Verified both are **active** in the dashboard.

📸 **Screenshot placeholders:**
- `screenshots/machines/dc01/25-dc01-wazuh-agent-setup.png`
- `screenshots/machines/wazuh/26-wazuh-both-agents-work.png`

### 9.3 Verify log flow
Confirmed Windows events arriving in Wazuh.

📸 **Screenshot placeholders:**
- `screenshots/machines/wazuh/27-wazuh-logs.png`

---

## 10. Sysmon → Wazuh Ingestion

Enabled Sysmon ingestion in `ossec.conf` using eventchannel localfile blocks, e.g.:
- `Security`
- `Microsoft-Windows-Sysmon/Operational`

Important note: channel names must be exact (typos cause `EvtSubscribe` errors).

📸 **Screenshot placeholders:**
- `screenshots/machines/win11/28-client01-wazuh-sysmon-setup-localfile.png`
- `screenshots/machines/wazuh/29-wazuh-sysmon-works.png`

---

## 11. ATTACK Network + Kali

Added a dedicated ATTACK segment:
- pfSense OPT2: `10.10.30.1/24`
- DHCP enabled on ATTACK
- Firewall rules allowing ATTACK → LAB-LAN
- Optional: block ATTACK → MGMT

Kali:
- DHCP IP: `10.10.30.100`
- Can reach pfSense and LAB-LAN targets after rules and Windows ICMP rules were adjusted.

📸 **Screenshot placeholders:**
- `screenshots/machines/pfsense/30-firewall-rules-attack.png`

---

## 12. VirtualBox Overview

All machines visible and connected via correct networks (WAN/LAB-LAN/MGMT/ATTACK).

📸 **Screenshot placeholders:**
- `screenshots/setup/33-overall-machine-setup-virtualbox.png`

---

# Troubleshooting Highlights

This lab included real-world issues and fixes that are worth documenting (and are valuable proof of skill):

1) **Windows host ICMP blocked** → enabled ICMP inbound rules  
📸 `screenshots/problems/04-host-firewall-enable-icmp.png`

2) **pfSense MGMT rules / WebUI access** → correct interface rules + allow WebConfigurator  
📸 `screenshots/machines/pfsense/07-rules-firewall-host-pfsense.png`

3) **Client got 169.254.x.x** → VirtualBox NIC + DHCP on pfSense LAN  
📸 `screenshots/machines/win11/11-client01-network-setup.png`

4) **Windows 11 Home cannot join a domain** → upgraded to Pro  
📸 `screenshots/machines/win11/13-client01-domain-change.png`

5) **DNS timeouts / forwarding strategy**
- validated with `nslookup`, Wazuh, and pfSense DNS handling  
📸 `screenshots/problems/20-dc01-nslookup-dns-times-out.png`  
📸 `screenshots/problems/21-client01-nslookup-works.png`

6) **DC01 default route / gateway issues** → enforced correct static gateway to pfSense  
📸 *(add your screenshot for the DC gateway fix here if you have one)*

7) **Wazuh Sysmon subscription errors (EvtSubscribe)**
- fixed by correct Sysmon install + correct event channel names  
📸 `screenshots/machines/win11/28-client01-wazuh-sysmon-setup-localfile.png`

---

# Next Steps: Purple-Team Scenarios

With the ATTACK network and SIEM telemetry working, the lab is ready for purple team loops:

**Planned scenarios**
- Password spray → detect (4625/4740) → lockout policy → re-test
- LLMNR/NBT-NS poisoning → detect → disable LLMNR via GPO → re-test
- Kerberoasting → detect (4769 patterns) → harden service accounts → re-test
- Lateral movement basics (SMB/WinRM) → detect → firewall/GPO hardening → re-test

Each scenario will be documented as:
- Attack steps
- Detection (Wazuh queries + screenshots)
- Hardening changes
- Validation (before/after)

---

## Notes
- This repository is for educational and lab purposes only.
- Feel free to contribute. No punishment for that :)
