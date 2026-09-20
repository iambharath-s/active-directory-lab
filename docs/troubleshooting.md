# Troubleshooting

Every error encountered during the build of this lab, with verified fixes.

---

## Network Issues

### VM gets 169.254.x.x instead of 192.168.56.x

**Cause:** The "Enable Network Adapter" checkbox in VirtualBox Settings → Network is unchecked,
even though the "Attached to" dropdown shows Host-only Adapter correctly.

**Fix:**
1. Power off the VM
2. Settings → Network → Adapter 1
3. Look at the very top of the tab — find the checkbox "Enable Network Adapter"
4. Verify it is **checked** (not greyed or unchecked)
5. Start the VM

---

### Kali eth0 has no IPv4 after reboot

**Fix — write permanent config:**
```bash
sudo nano /etc/network/interfaces
```

Replace contents with:
```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
    address 192.168.56.20
    netmask 255.255.255.0

auto eth1
iface eth1 inet dhcp
```

Then:
```bash
sudo systemctl restart networking
```

---

## DC01 Setup Issues

### New-ADUser fails: "password does not meet complexity requirements"

**Cause 1:** Default `MinPasswordAge = 1 day` blocks password resets on newly created accounts.

**Fix:** Run `03-password-policy.ps1` **before** running `05-create-users.ps1`:
```powershell
Set-ADDefaultDomainPasswordPolicy -Identity corp.lab -MinPasswordAge 0.00:00:00
```

**Cause 2:** Password contains `$` inside double quotes — PowerShell silently expands it as a variable.

**Fix:** Always use **single quotes** for passwords with `$` or special characters:
```powershell
# Wrong — $r gets expanded to empty string
ConvertTo-SecureString "Qw7$rTzv2024" -AsPlainText -Force

# Correct — single quotes prevent expansion
ConvertTo-SecureString 'Qw7#rTzv2024' -AsPlainText -Force
```

---

### setspn fails: "Unable to locate account svc-mssql"

**Cause:** `svc-mssql` was not created successfully (password failed during user creation).

**Fix — check the account state:**
```powershell
Get-ADUser svc-mssql -Properties Enabled, PasswordLastSet | Select Name, Enabled, PasswordLastSet
```

If `Enabled = False` and `PasswordLastSet` is empty:
```powershell
Set-ADAccountPassword -Identity "svc-mssql" -NewPassword (ConvertTo-SecureString 'MssqlSvc2024!' -AsPlainText -Force) -Reset
Enable-ADAccount -Identity "svc-mssql"
Set-ADUser -Identity "svc-mssql" -PasswordNeverExpires $true
```

Then re-run setspn:
```powershell
setspn -A MSSQLSvc/DC01.corp.lab:1433 CORP\svc-mssql
setspn -A MSSQLSvc/DC01:1433 CORP\svc-mssql
setspn -L CORP\svc-mssql
```

---

### LDAP anonymous bind returns "Operations error"

**Cause:** The `dsHeuristics` must be set on the actual AD Directory Service object —
**not** in the registry. Setting `HKLM:\...\NTDS\Parameters\DSHeuristics` does nothing for anonymous bind.

**Correct fix:**
```powershell
# Step 1: Set dSHeuristics on the AD object
Set-ADObject "CN=Directory Service,CN=Windows NT,CN=Services,CN=Configuration,DC=corp,DC=lab" `
    -Replace @{dSHeuristics="0000002"}

# Step 2: Grant ANONYMOUS LOGON read rights on domain root
$root = [ADSI]"LDAP://DC=corp,DC=lab"
$anon = New-Object System.Security.Principal.NTAccount("NT AUTHORITY", "ANONYMOUS LOGON")
$sid  = $anon.Translate([System.Security.Principal.SecurityIdentifier])
$rule = New-Object System.DirectoryServices.ActiveDirectoryAccessRule(
    $sid,
    [System.DirectoryServices.ActiveDirectoryRights]::GenericRead,
    [System.Security.AccessControl.AccessControlType]::Allow,
    [DirectoryServices.ActiveDirectorySecurityInheritance]::All
)
$root.psbase.ObjectSecurity.AddAccessRule($rule)
$root.psbase.CommitChanges()
```

No reboot required. Changes apply immediately.

---

### DNS zone transfer fails ("Transfer failed")

**Cause:** `SecureSecondaries` was set to `NoTransfer` which **blocks** zone transfers.
`NoTransfer` = deny all. `TransferAnyServer` = allow all. They do the opposite.

**Correct fix:**
```powershell
Set-DnsServerPrimaryZone -Name "corp.lab" -SecureSecondaries TransferAnyServer -Notify NoNotify
```

Verify:
```powershell
Get-DnsServerZone -Name "corp.lab" | Select ZoneName, SecureSecondaries
# Must show: TransferAnyServer
```

---

### gpp-decrypt returns "bad decrypt" error

**Cause:** The cpassword value is malformed or was not encrypted with the real GPP AES key.

This repo uses a verified correct cpassword:
```
DFo58CwlYMizGqNu0IyQQPemXgiR7wPmLYMy9GZFytw
```
Generated with the actual Microsoft GPP AES key. Decrypts to `Summer2024!`.

Test it:
```bash
gpp-decrypt DFo58CwlYMizGqNu0IyQQPemXgiR7wPmLYMy9GZFytw
# Must output: Summer2024!
```

---

### smbclient null session returns NT_STATUS_ACCESS_DENIED

**Cause:** One or more required registry keys is missing, OR Guest is still disabled,
OR a reboot has not occurred after the LSA changes.

**Complete fix (run all):**
```powershell
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "EveryoneIncludesAnonymous"  -Value 1 -Type DWord
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymous"           -Value 0 -Type DWord
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Control\Lsa" -Name "RestrictAnonymousSAM"        -Value 0 -Type DWord
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "RestrictNullSessAccess" -Value 0 -Type DWord
Set-ItemProperty "HKLM:\SYSTEM\CurrentControlSet\Services\LanmanServer\Parameters" -Name "NullSessionShares" -Value @("Data","Finance","IT_Admin","Backup$","NETLOGON","SYSVOL","IPC$") -Type MultiString
net user Guest /active:yes
Restart-Computer -Force
```

> LSA changes require a **full reboot** — a service restart alone is not enough.

---

### Meterpreter session opens then immediately dies

**Cause:** Windows Defender kills the meterpreter payload in memory.
Raw meterpreter is one of the most heavily signatured payloads.

**Fix — disable Defender before payload demos:**
```powershell
Set-MpPreference -DisableRealtimeMonitoring $true
Set-MpPreference -DisableIOAVProtection $true
Set-MpPreference -DisableBehaviorMonitoring $true
Add-MpPreference -ExclusionPath "C:\Temp\"
```

---

### AS-REP roasting returns no hash

```powershell
# Verify the flag is set
Get-ADUser john.doe -Properties DoesNotRequirePreAuth | Select Name, DoesNotRequirePreAuth
# Must show True

# Fix if False
Set-ADAccountControl john.doe -DoesNotRequirePreAuth $true
```

---

### Kerberoasting returns no SPN

```powershell
# Verify SPNs exist
setspn -L CORP\svc-mssql
# Must show: MSSQLSvc/DC01.corp.lab:1433 and MSSQLSvc/DC01:1433

# Fix if empty
setspn -A MSSQLSvc/DC01.corp.lab:1433 CORP\svc-mssql
setspn -A MSSQLSvc/DC01:1433 CORP\svc-mssql
```

---

## Known Non-Issues

**SMTP not included in this lab**

SMTP relay configuration via `inetmgr6` (IIS 6.0 Manager) consistently crashes on
Windows Server 2022 with a Snapin Error. The `root\MicrosoftIISv2` WMI namespace and
`adsutil.vbs` script are both absent from Server 2022. SMTP is excluded from this lab.

**Smurf attack shows limited amplification**

With only 2 VMs in the lab, Smurf produces minimal amplification.
In a real network with 500 hosts it would be 500x. The demo teaches the concept correctly
at small scale — limited amplification in the lab is expected behavior.
