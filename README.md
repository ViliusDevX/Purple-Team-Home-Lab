# Purple-Team-Home-Lab

A VirtualBox based Windows domain lab for practicing **IT support, Windows administration, troubleshooting, detection, and hardening** in a small enterprise style environment.

The goal of this project is to build hands on experience with real problems a junior IT support technician or junior sysadmin might face: user login issues, Group Policy problems, endpoint monitoring, Windows logs, network troubleshooting, and basic security hardening.

---

## Lab Environment

| Machine          | Purpose                                             |
| ---------------- | --------------------------------------------------- |
| `pfSense`        | Firewall and routing                                |
| `DC01`           | Windows Server / Active Directory Domain Controller |
| `WIN11-CLIENT01` | Domain joined Windows workstation                   |
| `WAZUH01`        | SIEM / endpoint monitoring                          |
| `KALI01`         | Controlled testing machine                          |

VirtualBox networks used:

* `LAB-WAN`
* `LAB-LAN`
* `Host-Only`
* `ATTACK`

---

## Repo Structure

```text
Purple-Team-Home-Lab/
│
├── setup-process/
│   ├── README.md
│   └── screenshots/
│
└── scenarios/
    ├── T1110-password-spray/
    └── T1557.001-llmnr-nbtns-poisoning/
```

* `setup-process/` documents how the lab was built.
* `scenarios/` contains practical troubleshooting, detection, hardening, and validation exercises.

---

## Completed Scenarios

| Scenario                            | Focus                                       | Skills Practiced                                                         |
| ----------------------------------- | ------------------------------------------- | ------------------------------------------------------------------------ |
| Password Spraying / Account Lockout | AD authentication and lockout investigation | Event Viewer, Wazuh, account lockout policy, Event ID 4625/4740          |
| LLMNR/NBT-NS Poisoning              | Name-resolution attack and hardening        | Responder, Sysmon Event ID 3, firewall rules, detection gaps, validation |

---

## Planned IT Support Scenarios

| Scenario                       | Focus                                                          |
| ------------------------------ | -------------------------------------------------------------- |
| GPO Not Applying               | `gpupdate`, `gpresult`, time sync, domain troubleshooting      |
| Network Share Access Issue     | AD groups, NTFS/share permissions, SMB, DNS                    |
| Endpoint Agent Disconnected    | Wazuh agent service, manager IP, re-registration, connectivity |
| Remote Support with PowerShell | WinRM, remote commands, firewall rules, admin access           |
| DNS/DHCP Troubleshooting       | pfSense, Windows DNS, DHCP leases, name resolution             |

---

## Skills Demonstrated

* Windows Server and Active Directory
* Windows 11 domain client administration
* Group Policy troubleshooting
* User/account lockout investigation
* DNS, DHCP, SMB, and firewall troubleshooting
* Event Viewer and Sysmon analysis
* Wazuh endpoint monitoring
* PowerShell troubleshooting
* Technical documentation and repeatable runbooks
* Security aware IT support practices

---

## Career Goal

This lab supports my goal of moving into a remote IT support or junior technical role, while building a foundation for future SOC/security operations work.

The focus is practical: reproduce problems, investigate them, fix them, validate the result, and document the process clearly.

---

## Disclaimer

This lab is for educational testing only. All testing is performed in an isolated lab environment on systems I own or have permission to use.
