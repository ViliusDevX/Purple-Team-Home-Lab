# Purple-Team-Home-Lab

A VirtualBox based purple team lab for practicing **attack simulation + detection + hardening** in a small, realistic Windows domain environment.

## What’s inside

**Lab machines**
- **pfSense**
- **DC01** (Windows Server AD DS)
- **WIN11-CLIENT01** (domain-joined workstation)
- **Wazuh** (agents + Sysmon)
- **Kali Linux**

**Networks (VirtualBox adapters)**
- `LAB-WAN` (NAT Network)
- `LAB-LAN` (Internal Network)
- `Host-Only`
- `ATTACK` (Internal Network)

## Repo contents

- `setup-process` - documentation of how i created this lab (includes screenshots folder)
- *More coming soon*

## Goals

- Build repeatable lab documentation (setup, configs, screenshots)
- Practice common enterprise attack paths (AD, endpoint, network)
- Improve visibility (Sysmon, Wazuh rules, firewall logging)
- Practice system hardening

## Disclaimer

This lab is for **educational testing** on systems you own or explicitly have permission to test.
