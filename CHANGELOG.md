# Changelog

All notable changes to active-directory-lab are documented here.
Format: [Version] - YYYY-MM-DD

---

## [1.0.0] - 2026-09-20

### Initial Release

**DC01 Attack Surfaces:**
- DNS zone transfer (AXFR) via TransferAnyServer
- LDAP anonymous bind via dSHeuristics + ANONYMOUS LOGON ACL
- AS-REP roasting via DoesNotRequirePreAuth on john.doe
- Kerberoasting via MSSQLSvc SPN on svc-mssql
- GPP cpassword in real SYSVOL (verified working cpassword hash)
- ACL misconfiguration: helpdesk GenericAll over ServiceAccounts OU
- AlwaysInstallElevated for privilege escalation demos
- SMB null sessions with 5 pre-populated shares
- SNMP public community string
- RDP with NLA disabled
- WinRM enabled

**Setup Scripts:**
- 8 numbered PowerShell scripts covering full DC01 configuration
- Kali permanent network configuration script

**Attack Scripts:**
- Enumeration chain (DNS, LDAP, SMB, SNMP)
- Kerberos attacks (AS-REP + Kerberoast + BloodHound)
- Lateral movement (PtH, evil-winrm, secretsdump)
- DoS/DDoS (SYN/UDP/ICMP/Smurf/Slowloris)
- Lab verification script (12 automated checks)

**Documentation:**
- Prerequisites guide
- Network setup guide
- Troubleshooting guide (all errors hit during build + fixes)
- Attack reference guide

**Known Issues:**
- SMTP relay via IIS6 not included — inetmgr6 crashes on Server 2022, no reliable configuration method available
- Smurf attack produces limited amplification in small lab (3 VMs) — expected behavior explained in docs

---

## Roadmap

- [ ] Windows 10/11 workstation VM (WS01) as domain-joined lateral movement target
- [ ] Zerologon (CVE-2020-1472) misconfiguration
- [ ] PrintNightmare (CVE-2021-1675) misconfiguration
- [ ] Constrained/Unconstrained delegation attack paths
- [ ] GitHub Actions CI to validate PowerShell script syntax
