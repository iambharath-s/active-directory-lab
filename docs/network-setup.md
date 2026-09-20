# Network Setup

## Overview

Both VMs connect through a VirtualBox Host-Only adapter.
This completely isolates all lab traffic from your physical network and the internet.

```
Your Physical Host  192.168.56.1
         |
VirtualBox Host-Only Network: 192.168.56.0/24
         |
    ┌────┴─────┐
    |          |
  DC01       KALI
192.168.56.10  192.168.56.20
```

---

## Step 1 — Create the Host-Only Network

1. Open VirtualBox
2. **File → Tools → Network Manager**
3. Click the **Host-only Networks** tab
4. Click **Create** (the + icon)
5. Select the new adapter and configure:
   - IPv4 Address: `192.168.56.1`
   - IPv4 Network Mask: `255.255.255.0`
6. Click **DHCP Server** tab → **uncheck Enable Server**
7. Click **Apply**

---

## Step 2 — Assign Network to DC01

**Settings → Network → Adapter 1:**

| Field | Value |
|-------|-------|
| ✅ Enable Network Adapter | **CHECKED** (verify the checkbox itself) |
| Attached to | Host-only Adapter |
| Name | VirtualBox Host-Only Ethernet Adapter |

> ⚠️ The most common failure: the "Enable Network Adapter" checkbox is unchecked
> even though the dropdowns are filled in correctly. The VM gets a 169.254.x.x
> APIPA address and nothing works. Always verify the checkbox visually.

Adapters 2, 3, 4: leave disabled.

---

## Step 3 — Assign Network to Kali

**Settings → Network → Adapter 1:**
- ✅ Enable Network Adapter
- Attached to: **Host-only Adapter**
- Name: same adapter as DC01

**Settings → Network → Adapter 2 (optional — for internet on Kali):**
- ✅ Enable Network Adapter
- Attached to: **NAT**

The NAT adapter gives Kali internet access for updates without exposing the lab.
DC01 has no internet adapter by design.

---

## Step 4 — Verify After Boot

**From DC01 PowerShell:**
```powershell
ipconfig
# Must show: IPv4 Address: 192.168.56.10
```

**From Kali terminal:**
```bash
ip addr show eth0
# Must show: 192.168.56.20/24

ping -c 2 192.168.56.10
# Must reply before running any setup scripts
```

---

## Network Isolation Check

Confirm no lab traffic reaches the internet:
```bash
# From Kali using eth0 only — should fail
curl --interface eth0 -m 5 https://google.com
# Expected: curl: (7) Couldn't connect to server
```

Only `eth1` (NAT) should have internet access on Kali.
