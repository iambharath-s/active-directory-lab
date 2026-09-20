# Prerequisites

## 1. Enable Virtualization in BIOS

Before anything else, hardware virtualization must be enabled.

Restart → enter BIOS (F2 / Del / F10 during boot):
- **Intel:** Intel VT-x (Virtualization Technology)
- **AMD:** AMD-V or SVM Mode

Enable, save, reboot.

Verify in Windows CMD:
```
systeminfo | findstr "Virtualization"
```
Expected: `A hypervisor has been detected.`

---

## 2. Install VirtualBox

Download both files from the **same version** at https://www.virtualbox.org/wiki/Downloads:

| File | Notes |
|------|-------|
| VirtualBox platform installer | Install this first |
| VirtualBox Extension Pack | Install after VirtualBox — double-click to install |

> ⚠️ Extension Pack version must exactly match VirtualBox version.

---

## 3. Download ISOs

Save both to a dedicated folder — e.g. `D:\Lab\ISOs\`

### Windows Server 2022 Evaluation (~5 GB)
URL: https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2022

1. Click "Start your evaluation"
2. Fill the form (any values — this is gated but free)
3. Select: **ISO, 64-bit**
4. Download

> 180-day evaluation. Extend with `slmgr /rearm` inside DC01 (up to 3 times).

### Kali Linux Installer 64-bit (~4 GB)
URL: https://www.kali.org/get-kali/#kali-installer-images

Select: **Installer** (not Live, not Virtual Machine) → 64-bit

---

## 4. Host Machine Requirements

| Resource | Minimum | Recommended |
|----------|---------|-------------|
| RAM | 8 GB | 16 GB |
| Disk free | 80 GB | 120 GB |
| CPU cores | 4 | 8 |
| CPU feature | VT-x or AMD-V | — |

### VM RAM Allocation

| VM | Allocated |
|----|-----------|
| DC01 | 4096 MB |
| KALI-ATK01 | 3072 MB |

If your host has only 8 GB total RAM: reduce DC01 to 2048 MB and Kali to 2048 MB.
The lab still works but will be slower.
