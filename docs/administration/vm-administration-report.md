# Linux VM Administration and Diagnostics Report

- **Student GitHub username:** talemwasharon
- **Issue:** #31
- **Date of inspection:** 2026-10-25
- **Environment:** Ubuntu desktop workstation (Dell OptiPlex Tower 7010)

> Hostname, IP address and personal details are sanitised as `<redacted>`.
> Sanitised command output is in `docs/administration/evidence/`.

## 1. System identification

| Item | Observed value | Command |
|---|---|---|
| Distribution | Ubuntu (ID_LIKE=debian) | `cat /etc/os-release` |
| Version | 26.04.1 LTS (Resolute Raccoon), VERSION_ID 26.04 | `cat /etc/os-release` |
| Kernel | 7.0.0-38-generic, x86_64 | `uname -a` |
| Hostname | `<redacted>` | `hostnamectl` |
| CPU | 13th Gen Intel Core i5-13500, 14 cores, 20 logical CPUs, 1 socket, max 4.8 GHz | `lscpu` |
| Memory | about 6.9 GiB total, 4.0 GiB swap | `free -h` |
| Storage | nvme0n1, 476.9G; root partition 104G | `df -h`, `lsblk` |
| Network | Ethernet `enp0s31f6`, UP, LOWER_UP, IPv4 `<redacted>/24` | `ip addr` |

## 2. Commands: purpose, output and interpretation

### 2.1 `hostnamectl`
**Purpose:** Shows the hostname, OS, kernel, architecture and whether the machine is physical or virtual.
**Observed:** See `evidence/hostnamectl.txt`. The hostname is `<redacted>`.
**Interpretation:** Identifies the machine and confirms the OS and kernel in use.

### 2.2 `cat /etc/os-release`
**Purpose:** Prints the distribution name and version.
**Observed:** Ubuntu, VERSION="26.04.1 LTS (Resolute Raccoon)", VERSION_ID="26.04", ID=ubuntu, ID_LIKE=debian.
**Interpretation:** The system is an Ubuntu LTS release, derived from Debian.

### 2.3 `uname -a`
**Purpose:** Shows the kernel release, architecture and hostname.
**Observed:** Kernel 7.0.0-38-generic, x86_64.
**Interpretation:** A 64-bit Linux kernel is running.

### 2.4 `lscpu`
**Purpose:** Lists CPU model, cores, threads and frequency.
**Observed:** Intel Core i5-13500 (13th Gen), 14 cores, 20 logical CPUs, 1 socket, max 4.8 GHz.
**Interpretation:** Processing resources are well above what is needed for normal use.

### 2.5 `free -h`
**Purpose:** Shows total, used, free and available memory and swap.
**Observed:** About 6.9 GiB total, 3.5 GiB used, 789 MiB free, about 3.0 GiB available; swap 4.0 GiB with 0 B used.
**Interpretation:** The "available" figure, not "free", shows what applications can use. See Scenario 2.

### 2.6 `df -h`
**Purpose:** Shows used and available space on each mounted filesystem.
**Observed:** Root filesystem 104G, 16G used, 84G available (16%). EFI partition 96M, 61M used, 36M available (63%).
**Interpretation:** No disk-space shortage. The EFI partition is small by design, and 63% is not a fault.

### 2.7 `lsblk`
**Purpose:** Shows disks, partitions and mount points.
**Observed:** nvme0n1 (476.9G) with a 100M EFI partition at `/boot/efi` and a 105.8G root partition at `/`. Several loop devices under `/snap/`.
**Interpretation:** No storage fault. The loop devices are normal for Snap packages.

### 2.8 `ip addr`
**Purpose:** Shows network interfaces, state and IP addresses.
**Observed:** `lo` at 127.0.0.1; `enp0s31f6` UP with LOWER_UP and an IPv4 address in a /24 network (`<redacted>`).
**Interpretation:** The Ethernet link is detected and the interface is up.

### 2.9 `systemctl --failed`
**Purpose:** Lists systemd units in a failed state.
**Observed:** 0 loaded units listed.
**Interpretation:** No failed units at that moment. This does not prove every service works correctly.

### 2.10 `journalctl -b -p warning --no-pager`
**Purpose:** Shows warning-level and more severe log messages from the current boot.
**Observed:** USB initialisation and enumeration errors; a warning from `ubuntu-insights.service`; an assertion message from GNOME Files (Nautilus).
**Interpretation:** These are observations needing further checking, not automatically faults. The USB errors are examined in Scenario 1.

## 3. Diagnostic scenario 1: USB device initialisation errors

- **Symptoms:** Repeated USB disconnections, failures to enable the port and port power-cycling attempts. A phone was connected at the time.
- **Evidence:** `journalctl -b -k --no-pager | grep -iE 'usb|error -71|enumerate'` (sanitised output in `evidence/usb-kernel.txt`).
- **Analysis:** The kernel had difficulty communicating with and initialising the connected device. The repeated errors suggest an intermittent connection or initialisation problem.
- **Likely cause (not confirmed):** A faulty or loose cable, a problem with the phone's port, or a problem with the computer's USB port.
- **Status:** The errors are a **confirmed observation** in the kernel log. The underlying hardware cause is an **assumption**.
- **Recommended action:** Check the cable and connections, test with a known-good cable and a different USB port, and see whether the errors recur. Make no configuration changes without further evidence.

## 4. Diagnostic scenario 2: Possible memory pressure

- **Symptoms:** None reported. Checked as a routine investigation.
- **Evidence:** `free -h` showed about 3.0 GiB available and 0 B of swap used.
- **Analysis:** The low "free" value (789 MiB) is normal, because Linux uses spare RAM for caching.
- **Status:** **No fault found.** This is a single snapshot and does not rule out shortages under heavier workloads.
- **Recommended action:** None needed now. If slowness appears, check again under load with `free -h` and `top`.

## 5. Summary of findings

| Finding | Category | Evidence |
|---|---|---|
| USB communication and initialisation errors | Confirmed observation; cause unconfirmed | Kernel log |
| Memory pressure | No fault found | `free -h` |
| Disk space | No fault found | `df -h`, `lsblk` |
| Failed services | None reported | `systemctl --failed` |
| `ubuntu-insights.service` warning, Nautilus assertion | Unverified, needs further investigation | `journalctl` |

## 6. Safety and privacy statement

No services were disabled and no persistent system changes were made. Only read-only commands were run. Evidence has been checked to contain no passwords, tokens, private keys or personal information.


