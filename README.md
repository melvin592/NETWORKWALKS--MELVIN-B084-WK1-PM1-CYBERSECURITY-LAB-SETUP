# Kali Linux in VirtualBox with NAT Network — Detailed Step-by-Step Guide

> A complete, professional walkthrough for building an isolated, reproducible Kali Linux lab for **Red Team** and **Blue Team** practice. Every step includes exact menus, commands, and expected results.

---

## Table of Contents

1. [Overview](#overview)
2. [Why a Professional Installation Matters](#why-a-professional-installation-matters)
3. [Prerequisites & Hardware Check](#prerequisites--hardware-check)
4. [Part 1 — Install VirtualBox (Detailed)](#part-1--install-virtualbox-detailed)
5. [Part 2 — Download & Verify Kali Linux](#part-2--download--verify-kali-linux)
6. [Part 3 — Create the Virtual Machine (Detailed)](#part-3--create-the-virtual-machine-detailed)
7. [Part 4 — Install Kali Linux (Detailed)](#part-4--install-kali-linux-detailed)
8. [Part 5 — First Boot & System Update](#part-5--first-boot--system-update)
9. [Part 6 — Install Guest Additions](#part-6--install-guest-additions)
10. [Part 7 — Configure the NAT Network](#part-7--configure-the-nat-network)
11. [Part 8 — Configure Port Forwarding (Optional)](#part-8--configure-port-forwarding-optional)
12. [Part 9 — Verify Everything Works](#part-9--verify-everything-works)
13. [Part 10 — Create Your Baseline Snapshot](#part-10--create-your-baseline-snapshot)
14. [Benefits for Red Team](#benefits-for-red-team)
15. [Benefits for Blue Team](#benefits-for-blue-team)
16. [Operational Best Practices](#operational-best-practices)
17. [Troubleshooting](#troubleshooting)
18. [Legal & Ethical Notice](#legal--ethical-notice)

---

## Overview

This guide installs **Kali Linux** as a virtual machine inside **Oracle VirtualBox**, connected through a **VirtualBox NAT Network**. The environment is isolated from your real network, supports multiple VMs talking to each other, and can be rolled back instantly using snapshots.

**What you will end up with:**

- A working Kali VM with internet access
- A private NAT Network (`10.0.2.0/24`)
- Guest Additions (clipboard, drag-and-drop, resizing)
- A clean baseline snapshot
- A documented, repeatable lab

**Time required:** 45–90 minutes.

---

## Why a Professional Installation Matters

| Benefit | Why It Matters |
|---|---|
| **Isolation** | A NAT Network keeps attack traffic off your home/office LAN. |
| **Reproducibility** | Documented steps + snapshots let you rebuild in minutes. |
| **Safety** | Exploitation, scanning, and malware analysis stay contained. |
| **Credibility** | Clean configuration mirrors professional lab standards. |
| **Maintainability** | Snapshots, backups, and update discipline prevent downtime. |
| **Auditability** | A documented environment supports responsible testing. |

A professional build is about **control, safety, and repeatability** — not appearance.

---

## Prerequisites & Hardware Check

### Hardware requirements

| Resource | Minimum | Recommended |
|---|---|---|
| CPU | 2 cores | 4+ cores |
| RAM | 8 GB host | 16 GB host |
| Disk | 40 GB free | 60+ GB SSD |
| Network | Any | Wired preferred |

### Check virtualization support

**Windows (PowerShell — run as Administrator):**

```powershell
systeminfo | Select-String "Virtualization"
```

Look for:

```text
Virtualization Enabled In Firmware: Yes
```

Or use Task Manager → **Performance → CPU** and check **Virtualization: Enabled**.

**Linux:**

```bash
grep -Eoc '(vmx|svm)' /proc/cpuinfo
```

A result greater than `0` means virtualization is available.

**macOS:**

```bash
sysctl -a | grep machdep.cpu.features | grep VMX
```

### Enable virtualization in BIOS/UEFI (if disabled)

1. Reboot and enter BIOS/UEFI (usually `F2`, `F10`, `Del`, or `Esc` — depends on vendor).
2. Find **CPU Configuration** or **Advanced → CPU**.
3. Enable **Intel VT-x** / **Intel Virtualization Technology** or **AMD-V** / **SVM Mode**.
4. Save and exit (`F10`).

> ⚠️ On Windows, disabling **Hyper-V** may be required if VirtualBox reports VT-x is unavailable. Run `bcdedit /set hypervisorlaunchtype off` and reboot (only if needed).

---

## Part 1 — Install VirtualBox (Detailed)

### Step 1.1 — Download VirtualBox

1. Open your browser and go to [https://www.virtualbox.org/wiki/Downloads](https://www.virtualbox.org/wiki/Downloads).
2. Under **VirtualBox Platform Packages**, click the installer for your host:
   - **Windows hosts**
   - **macOS / Intel hosts** or **macOS / Apple Silicon hosts**
   - **Linux distributions** (choose your distro)

### Step 1.2 — Download the Extension Pack

1. On the same page, under **VirtualBox Extension Pack**, download **All supported platforms**.
2. Save it — it will have a `.vbox-extpack` extension.

### Step 1.3 — Install VirtualBox

**Windows:**

1. Double-click the `.exe` installer.
2. Click **Next** on the welcome screen.
3. Keep the default install location → **Next**.
4. On the **Custom Setup** screen, keep all features enabled → **Next**.
5. Click **Yes** on the network interface warning → **Next**.
6. Click **Install** and allow the driver installation prompt.
7. Click **Finish** to launch VirtualBox.

**macOS:**

1. Open the `.dmg` file.
2. Double-click **VirtualBox.pkg** → **Continue** → **Install**.
3. Approve the security prompt in **System Settings → Privacy & Security** if macOS blocks it.

**Linux (Debian/Ubuntu):**

```bash
sudo apt update
sudo apt install -y virtualbox virtualbox-ext-pack
```

### Step 1.4 — Install the Extension Pack

1. Launch **VirtualBox**.
2. Go to **File → Preferences → Extensions**.
3. Click the **+** icon on the right.
4. Select the downloaded `.vbox-extpack` file → **Open**.
5. Scroll to the bottom of the license → click **I Agree**.
6. Enter your system password if prompted.

### Step 1.5 — Verify the installation

Open a terminal:

```bash
VBoxManage --version
```

Expected output (version may differ):

```text
7.1.6r167084
```

> ✅ **Checkpoint:** VirtualBox and the Extension Pack versions must match. If they differ, download the matching Extension Pack.

---

## Part 2 — Download & Verify Kali Linux

### Step 2.1 — Download the ISO

1. Go to [https://www.kali.org/get-kali/](https://www.kali.org/get-kali/).
2. Scroll to **Installer Images**.
3. Click **Download** under the **64-bit** installer.
4. Save the ISO (e.g., `kali-linux-2025.1-installer-amd64.iso`) to a known folder.

> 💡 Use the **Installer** image (not the Live image) for a permanent VM installation.

### Step 2.2 — Verify the SHA-256 checksum

1. On the Kali download page, click **SHA256SUMS** to view the official hash.
2. Compute the hash of your downloaded file.

**Windows (PowerShell):**

```powershell
Get-FileHash .\kali-linux-2025.1-installer-amd64.iso -Algorithm SHA256
```

**Linux:**

```bash
sha256sum kali-linux-2025.1-installer-amd64.iso
```

**macOS:**

```bash
shasum -a 256 kali-linux-2025.1-installer-amd64.iso
```

3. Compare the output with the official hash. **They must match exactly.**

> ✅ **Checkpoint:** Never boot an unverified ISO. A mismatch means the download is corrupted or tampered with.

---

## Part 3 — Create the Virtual Machine (Detailed)

### Step 3.1 — Open the New VM wizard

1. Launch **VirtualBox**.
2. Click **New** (or **Machine → New**).

### Step 3.2 — Name and operating system

| Field | Value |
|---|---|
| **Name** | `Kali-Linux-Lab` |
| **Folder** | Default (or a dedicated VM folder) |
| **ISO Image** | Leave blank for now (we attach it manually) |
| **Type** | Linux |
| **Version** | Debian (64-bit) |

Click **Next**.

### Step 3.3 — Hardware

| Field | Value | Notes |
|---|---|---|
| **Base Memory** | `4096 MB` | Use 50% of host RAM if host has 8 GB |
| **Processors** | `2`–`4` | Do not exceed host physical cores |

Click **Next**.

### Step 3.4 — Virtual hard disk

1. Select **Create a Virtual Hard Disk Now**.
2. Set **Disk Size** to `40.00 GB`.
3. Click **Next**.

### Step 3.5 — Review and finish

1. Review the summary.
2. Click **Finish**.

### Step 3.6 — Tune VM settings before first boot

Select the VM → click **Settings**.

**System → Motherboard:**

| Setting | Value |
|---|---|
| **Base Memory** | 4096 MB |
| **Boot Order** | Hard Disk, Optical, Network |
| **Chipset** | PIIX3 (default) |

**System → Processor:**

| Setting | Value |
|---|---|
| **Processors** | 2–4 |
| **Enable Nested VT-x/AMD-V** | Optional (only for nested virtualization labs) |

**Display → Screen:**

| Setting | Value |
|---|---|
| **Video Memory** | 128 MB |
| **Graphics Controller** | VMSVGA |
| **Enable 3D Acceleration** | Optional (can cause issues on some hosts) |

**Storage:**

1. Under **Controller: IDE**, select the **Empty** optical drive.
2. Click the **disk icon** on the right → **Choose a disk file…**
3. Select your verified Kali ISO → **Open**.

**Network → Adapter 1:**

| Setting | Value |
|---|---|
| **Enable Network Adapter** | ✅ Checked |
| **Attached to** | NAT Network (configure it in Part 7 first, or leave NAT for now) |
| **Name** | `KaliNatNet` (once created) |

Click **OK**.

> ✅ **Checkpoint:** The ISO is mounted, RAM/CPU/disk are set, and the network adapter is enabled.

---

## Part 4 — Install Kali Linux (Detailed)

### Step 4.1 — Start the VM

1. Select `Kali-Linux-Lab` → click **Start** (green arrow).
2. A new window opens and boots from the ISO.

### Step 4.2 — Boot menu

1. At the Kali boot menu, use the arrow keys to select **Graphical Install**.
2. Press **Enter**.

### Step 4.3 — Language and location

1. **Language:** select your language → **Continue**.
2. **Location:** select your country → **Continue**.
3. **Keyboard:** select your layout → **Continue**.

### Step 4.4 — Network configuration

1. **Hostname:** enter `kali-lab` → **Continue**.
2. **Domain name:** leave blank or enter `lab.local` → **Continue**.

### Step 4.5 — User account

1. **Root password:** enter a strong password → **Continue** → re-enter → **Continue**.
2. **Full name:** e.g., `Kali User` → **Continue**.
3. **Username:** e.g., `kali` → **Continue**.
4. **User password:** enter a strong password → **Continue** → re-enter → **Continue**.

> 🔐 Use different, strong passwords for root and the daily user. Never reuse real-world credentials.

### Step 4.6 — Clock and time zone

1. Select your time zone → **Continue**.

### Step 4.7 — Disk partitioning

1. Choose **Guided — use entire disk** → **Continue**.
2. Select the virtual disk (only one should be listed) → **Continue**.
3. Choose **All files in one partition (recommended for new users)** → **Continue**.
4. Review the partition table → select **Finish partitioning and write changes to disk** → **Continue**.
5. Select **Yes** to write changes → **Continue**.

### Step 4.8 — Package manager and mirrors

1. **Use a network mirror?** → select **Yes** → **Continue**.
2. **Mirror country:** select yours → **Continue**.
3. **Mirror host:** keep the default → **Continue**.
4. **HTTP proxy:** leave blank → **Continue**.

### Step 4.9 — Software selection

1. Keep the default **Debian desktop environment** and **XFCE** checked.
2. Optionally check **standard system utilities**.
3. Click **Continue**.

> 💡 XFCE is lightweight and stable — ideal for a VM.

### Step 4.10 — Install GRUB

1. When prompted, choose **Yes** to install GRUB to the master boot record → **Continue**.
2. Select the virtual disk `/dev/sda` → **Continue**.

### Step 4.11 — Finish installation

1. Installation completes → click **Continue** to reboot.
2. If the VM reboots back into the installer, go to **Devices → Optical Drives → Remove Disk from Virtual Drive**, then reboot again.

### Step 4.12 — First login

1. At the login screen, select your user and enter the password.
2. You should land on the XFCE desktop.

> ✅ **Checkpoint:** Kali boots to the desktop, and you can open a terminal.

---

## Part 5 — First Boot & System Update

### Step 5.1 — Open a terminal

Click the terminal icon in the taskbar, or press `Ctrl+Alt+T`.

### Step 5.2 — Update package lists and upgrade

```bash
sudo apt update
sudo apt full-upgrade -y
```

- `apt update` refreshes the package index.
- `apt full-upgrade` upgrades all packages and handles dependency changes.

### Step 5.3 — Clean up

```bash
sudo apt autoremove -y
sudo apt autoclean
```

### Step 5.4 — Reboot

```bash
sudo reboot
```

### Step 5.5 — Verify the version

```bash
cat /etc/os-release
uname -r
```

> ✅ **Checkpoint:** The system is fully updated and running the latest kernel.

---

## Part 6 — Install Guest Additions

Guest Additions enable clipboard sharing, drag-and-drop, automatic display resizing, and better mouse integration.

### Step 6.1 — Install build dependencies

```bash
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
```

### Step 6.2 — Insert the Guest Additions CD

1. In the VirtualBox menu, click **Devices → Insert Guest Additions CD image…**
2. Click **Cancel** if the file manager prompt appears.

### Step 6.3 — Mount and run the installer

```bash
sudo mkdir -p /mnt/cdrom
sudo mount /dev/cdrom /mnt/cdrom
cd /mnt/cdrom
sudo ./VBoxLinuxAdditions.run
```

### Step 6.4 — Reboot

```bash
sudo reboot
```

### Step 6.5 — Alternative: distro package

If the manual method fails, install the packaged version:

```bash
sudo apt install -y virtualbox-guest-x11
sudo reboot
```

### Step 6.6 — Enable clipboard and drag-and-drop

1. In the VM window menu: **Devices → Shared Clipboard → Bidirectional**.
2. **Devices → Drag and Drop → Bidirectional**.

### Step 6.7 — Test

1. Copy text on the host → paste inside the VM.
2. Resize the VM window → the desktop should auto-resize.

> ✅ **Checkpoint:** Clipboard sharing and display resizing work.

---

## Part 7 — Configure the NAT Network

### Step 7.1 — Open the Network Manager

1. In the VirtualBox main window: **File → Tools → Network Manager**.
2. Select the **NAT Networks** tab.

### Step 7.2 — Create a NAT Network

1. Click **Create** (or **+**).
2. Configure:

| Field | Value |
|---|---|
| **Name** | `KaliNatNet` |
| **Network CIDR** | `10.0.2.0/24` |
| **DHCP** | ✅ Enabled |
| **IPv6** | ❌ Disabled (for simplicity) |
| **Port Forwarding** | Empty (add later if needed) |

3. Click **Apply** (or **OK**).

### Step 7.3 — Attach the VM to the NAT Network

1. Select the `Kali-Linux-Lab` VM → **Settings → Network**.
2. **Adapter 1:**
   - **Enable Network Adapter:** ✅
   - **Attached to:** `NAT Network`
   - **Name:** `KaliNatNet`
3. Click **OK**.

### Step 7.4 — (Optional) Add a Host-Only adapter for management

1. **Settings → Network → Adapter 2.**
2. **Enable Network Adapter:** ✅
3. **Attached to:** `Host-only Adapter`
4. **Name:** `vboxnet0` (create it via **File → Host Network Manager** if missing).
5. Click **OK**.

> 💡 Adapter 1 (NAT Network) provides internet and VM-to-VM traffic. Adapter 2 (Host-Only) lets your host manage the VM without exposing it.

### Step 7.5 — Why NAT Network? (comparison)

| Mode | Internet | Talks to LAN | Isolation | Best For |
|---|---|---|---|---|
| **NAT** | ✅ | ❌ | High | Simple single-VM use |
| **NAT Network** | ✅ | ❌ | High | **Multi-VM labs** |
| **Bridged** | ✅ | ✅ | Low | Simulating a LAN device |
| **Host-Only** | ❌ | Host only | Highest | Fully isolated internal labs |
| **Internal Network** | ❌ | VMs only | Highest | Air-gapped multi-VM labs |

> ⚠️ Avoid **Bridged** mode for lab VMs unless you fully understand the exposure — it puts the VM directly on your real LAN.

---

## Part 8 — Configure Port Forwarding (Optional)

Use this only if you need to reach a service on the Kali VM from the host (e.g., SSH).

### Step 8.1 — Add a port-forwarding rule

1. **File → Tools → Network Manager → NAT Networks.**
2. Select `KaliNatNet` → click **Port Forwarding** (or the gear icon).
3. Click **Add** and configure:

| Rule Name | Protocol | Host IP | Host Port | Guest IP | Guest Port |
|---|---|---|---|---|---|
| SSH | TCP | `127.0.0.1` | `2222` | `10.0.2.15` | `22` |

4. Click **OK**.

> 🔒 Bind to `127.0.0.1`, **not** `0.0.0.0`, to keep the service host-only.

### Step 8.2 — Enable and start SSH in Kali

```bash
sudo systemctl enable --now ssh
sudo systemctl status ssh
```

### Step 8.3 — Connect from the host

```bash
ssh -p 2222 kali@127.0.0.1
```

> ✅ **Checkpoint:** The SSH prompt appears, and you can log in with your Kali user password.

---

## Part 9 — Verify Everything Works

Run these inside the Kali VM.

### Step 9.1 — Check the IP address

```bash
ip a
```

Expected: an interface with an address in `10.0.2.0/24` (e.g., `10.0.2.15`).

### Step 9.2 — Check the default route

```bash
ip route
```

Expected: `default via 10.0.2.1 dev eth0`.

### Step 9.3 — Test internet connectivity

```bash
ping -c 3 1.1.1.1
ping -c 3 kali.org
```

Expected: replies from both. If the IP ping works but the domain fails, DNS is misconfigured.

### Step 9.4 — Test DNS resolution

```bash
nslookup kali.org
```

Expected: a resolved IP address.

### Step 9.5 — Fix common network issues

If no IP is assigned:

```bash
sudo dhclient
```

If the network service is stuck:

```bash
sudo systemctl restart NetworkManager
```

If DNS fails, check `/etc/resolv.conf`:

```bash
cat /etc/resolv.conf
```

> ✅ **Checkpoint:** Internet, DNS, and routing all work.

---

## Part 10 — Create Your Baseline Snapshot

### Step 10.1 — Take a snapshot

1. Select the VM → **Machine → Take Snapshot** (or `Ctrl+Shift+S`).
2. Name it `Baseline — Clean Install + Updates`.
3. Add a description: `Kali 2025.x, updated, Guest Additions, NAT Network configured.`
4. Click **OK**.

### Step 10.2 — Verify the snapshot

1. Select the VM → click **Snapshots** in the right pane.
2. Confirm the snapshot appears.

### Step 10.3 — Practice restoring

1. Right-click the snapshot → **Restore Snapshot**.
2. Confirm — the VM returns to the exact saved state.

> ✅ **Checkpoint:** You can now roll back any experiment in seconds. Take a new snapshot before every risky scenario.

---

## Benefits for Red Team

| Capability | How the Setup Helps |
|---|---|
| **Safe exploitation** | NAT Network isolation prevents accidental attacks on real networks. |
| **Multi-VM attack chains** | Run a target VM (DVWA, Metasploitable 2, Windows AD lab) on the same NAT Network. |
| **Realistic pivoting** | NAT + Host-Only adapters simulate enterprise segmentation. |
| **Rapid resets** | Snapshots restore a compromised VM to a clean state in seconds. |
| **Tooling flexibility** | Full Kali toolset with internet access for updates and payload retrieval. |
| **Listener/C2 testing** | Controlled port forwarding enables lab C2 servers. |
| **Reproducible reporting** | Findings can be re-verified on an identical environment. |

**Red Team topology:**

```text
        ┌──────────────┐
        │   Host OS    │
        └──────┬───────┘
               │ NAT Network (10.0.2.0/24)
     ┌─────────┴─────────┐
┌────▼────┐        ┌─────▼─────┐
│  Kali   │◄──────►│  Target   │
│ Attacker│        │ (DVWA /   │
│  VM     │        │ Metasploit│
└─────────┘        │ able 2)   │
                   └───────────┘
```

---

## Benefits for Blue Team

| Capability | How the Setup Helps |
|---|---|
| **Detection engineering** | Generate attack traffic in isolation and validate IDS/IPS and SIEM rules. |
| **Log analysis** | Collect and analyze logs without contaminating production. |
| **Malware sandboxing** | Analyze samples in a contained, snapshot-backed VM. |
| **Incident response drills** | Practice triage and containment on a disposable environment. |
| **Forensics training** | Capture memory/disk artifacts against a known-good baseline. |
| **Threat hunting** | Simulate attacker TTPs and tune detections (MITRE ATT&CK mapping). |
| **Safe experimentation** | Break things freely — restore a snapshot and continue. |

**Blue Team topology:**

```text
┌──────────────┐   NAT Network   ┌────────────────┐
│  Analyst VM  │◄───────────────►│  Target VM +   │
│  (Kali)      │                 │  SIEM / IDS    │
└──────────────┘                 └────────────────┘
        │
        └── Snapshot before every scenario
```

---

## Operational Best Practices

1. **Snapshot before every risky activity.** Use clear names (`Pre-<scenario>`).
2. **Keep the host clean.** Never install offensive tooling on the host OS.
3. **Use a separate lab user** and never reuse real credentials.
4. **Update quarterly** and re-snapshot after updates.
5. **Document the lab.** Track VM names, IPs, credentials, and scenario notes.
6. **Match VirtualBox and Extension Pack versions.**
7. **Back up important snapshots** to external storage.
8. **Disable shared folders** unless explicitly needed.
9. **Use Host-Only adapters for management**, not broad exposure.
10. **Never use Bridged mode** on a lab VM without understanding the exposure.

---

## Troubleshooting

| Problem | Likely Cause | Fix |
|---|---|---|
| VM won't boot / black screen | Virtualization disabled | Enable VT-x/AMD-V in BIOS |
| "VT-x is not available" | Hyper-V enabled | `bcdedit /set hypervisorlaunchtype off` + reboot |
| No internet in VM | Adapter not on NAT Network | Check Adapter 1 → NAT Network |
| No IP address | DHCP not running | `sudo dhclient` |
| DNS fails but IP ping works | Resolver misconfigured | Check `/etc/resolv.conf` |
| Guest Additions fail | Missing headers | `sudo apt install linux-headers-$(uname -r)` |
| Clipboard not working | Guest Additions missing | Reinstall Guest Additions |
| Slow performance | Insufficient RAM/vCPUs | Allocate more resources |
| USB devices not detected | Extension Pack missing | Install matching Extension Pack |
| Port forwarding fails | Wrong guest IP | Verify with `ip a` |
| Installer loops on reboot | ISO still mounted | Remove ISO from virtual drive |
| GRUB fails to install | Wrong disk selected | Re-run installer, select `/dev/sda` |

---

## Legal & Ethical Notice

> **Use this lab only for lawful, authorized purposes.**
>
> - Test only systems you own or have **explicit written permission** to test.
> - Do not use lab knowledge or tooling against production systems without authorization.
> - Follow your organization's policies and applicable laws (e.g., CFAA, GDPR, local computer-misuse statutes).
> - Respect software licenses and terms of service.

---

## References

- [Kali Linux Documentation](https://www.kali.org/docs/)
- [VirtualBox User Manual](https://www.virtualbox.org/manual/)
- [VirtualBox Network Modes](https://www.virtualbox.org/manual/ch06.html)
- [MITRE ATT&CK](https://attack.mitre.org/)
- [OWASP Testing Guide](https://owasp.org/www-project-web-security-testing-guide/)

lines 1-758
