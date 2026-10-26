# Linux VM Administration and Diagnostics Report

- **Student GitHub username:** BbulyaAgatha
- **Assigned Issue:** [#26 — Network and Service Health Diagnostics](https://github.com/bos-com/bugema-linux/issues/26)
- **Practical:** [STUDENT PRACTICAL] Linux VM Administration and Diagnostics Report
- **Week:** 1 — Linux administration and diagnostics
- **Work area:** Administration
- **Submission deadline:** 26 October 2026
- **Date of inspection:** 26 October 2026

## 1. Objective

This report documents the practical Linux administration task in which a Linux
virtual machine was inspected, system information was collected, and at least two
safe diagnostic investigations were performed.

The investigation used read-only commands only. No service was disabled, no
essential system component was stopped, and no persistent system changes were made.

## 2. System identification

The following values were collected from the inspected Linux system.

| Item | Observed value | Command |
|---|---|---|
| Distribution | Ubuntu, `ID=ubuntu`, `ID_LIKE=debian` | `cat /etc/os-release` |
| Version | 24.04.4 LTS (Noble Numbat), `VERSION_ID="24.04"` | `cat /etc/os-release` |
| Kernel | `7.0.0-31-generic` (`#31~24.04.1-Ubuntu SMP ...`), x86_64 | `uname -a` |
| Hostname | `<redacted>` | `hostnamectl` |
| Architecture | x86-64 | `uname -a`, `lscpu` |
| CPU | GenuineIntel Core i5-8350U CPU @ 1.70GHz; 8 CPUs (4 cores, 2 threads per core), 1 socket; max frequency 3.6 GHz; `VT-x` virtualization support | `lscpu` |
| Memory | 7.6 GiB total, 5.7 GiB used, 580 MiB free, 2.2 GiB buff/cache, 1.9 GiB available; swap 4.0 GiB total | `free -h` |
| Storage | Disk `/dev/sda` 238.5 GiB (LVM); root logical volume 231 GiB, 74 GiB used, 146 GiB available (34%); `/boot` 2.0 GiB; `/boot/efi` 1.1 GiB | `df -h`, `lsblk` |
| Network | `lo` UP; `enp0s31f6` Ethernet UP, LOWER_UP; `wlp2s0` Wi-Fi UP, LOWER_UP; `docker0` DOWN | `ip addr` |

> **Note:** The inspected system reports Ubuntu 24.04.4 LTS, whereas ADR-001 of the
> Bugema Linux project targets Ubuntu 26.04 LTS. This discrepancy should be verified
> against the lab VM image before drawing conclusions about the project-specific
> environment.

## 3. Commands: purpose, output and interpretation

### 3.1 `hostnamectl`

**Purpose:** Display the static and pretty hostname, chassis type, operating system,
kernel, architecture and general system context.

**Observed:**

```text
 Static hostname: <redacted>
       Pretty hostname: <redacted>
       Icon name: computer-laptop
         Chassis: laptop
...
 Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 7.0.0-31-generic
    Architecture: x86-64
 Hardware Vendor: Dell Inc.
  Hardware Model: Latitude 7390
...
```

**Interpretation:** The system identity and OS/kernel information were confirmed by
`hostnamectl`. Full output: [`evidence/hostnamectl.txt`](evidence/hostnamectl.txt).

### 3.2 `cat /etc/os-release`

**Purpose:** Print the distribution name, version and base distribution.

**Observed:**

```text
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian
...
UBUNTU_CODENAME=noble
```

**Interpretation:** The system is an Ubuntu LTS release (Noble Numbat) derived from
Debian. Full output: [`evidence/os-release.txt`](evidence/os-release.txt).

### 3.3 `uname -a`

**Purpose:** Print kernel name, kernel release, kernel version, architecture, hostname
and other system identifiers.

**Observed:**

```text
Linux <redacted> 7.0.0-31-generic #31~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Aug 10 09:38:02 UTC 2 x86_64 x86_64 GNU/Linux
```

**Interpretation:** A 64-bit (`x86_64`) Linux kernel is running. Full output:
[`evidence/uname-a.txt`](evidence/uname-a.txt).

### 3.4 `lscpu`

**Purpose:** Report CPU architecture, model, core/thread counts, frequencies, cache
sizes and NUMA topology.

**Observed:**

```text
Architecture: x86_64
CPU op-mode(s): 32-bit, 64-bit
...
CPU(s): 8
On-line CPU(s) list: 0-7
Vendor ID: GenuineIntel
Model name: Intel(R) Core(TM) i5-8350U CPU @ 1.70GHz
...
Virtualization: VT-x
...
NUMA node(s): 1
NUMA node0 CPU(s): 0-7
```

**Interpretation:** An Intel Core i5-8350U dual-core/eight-thread processor with
virtualization support (`VT-x`) is available, which is suitable for running virtual
machines.

### 3.5 `free -h`

**Purpose:** Report total, used, free and available memory (human-readable) plus swap.

**Observed:**

```text
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       5.7Gi       580Mi       661Mi       2.2Gi       1.9Gi
Swap:          4.0Gi       3.3Gi       677Mi
```

**Interpretation:** About 1.9 GiB of memory is available to applications; about 2.2 GiB
is used as buff/cache, which is normal Linux behaviour and does not necessarily
indicate pressure.

### 3.6 `df -h`

**Purpose:** Report filesystem usage in human-readable form.

**Observed:**

```text
Filesystem                         Size  Used Avail Use% Mounted on
tmpfs                              782M  3.6M  778M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv  231G  74G  146G  34% /
...
/dev/sda2                          2.0G  332M  1.5G   19% /boot
/dev/sda1                          1.1G  6.2M  1.1G    1% /boot/efi
```

**Interpretation:** The root filesystem uses 34% of its capacity; no disk-space
shortage is present.

### 3.7 `lsblk`

**Purpose:** Report block device hierarchy, partitions and mount points.

**Observed:**

```text
sda                           8:0    0 238.5G  0 disk
├─sda1                        8:1    0     1G  0 part  /boot/efi
├─sda2                        8:2    0     2G  0 part  /boot
└─sda3                        8:3    0 235.4G  0 part
  └─dm_crypt-0              252:0    0 235.4G  0 crypt
    └─ubuntu--vg-ubuntu--lv 252:1    0 235.4G  0 lvm /
...
```

**Interpretation:** Standard partitioning with LVM and encrypted root volume plus
loop devices used by Snap packages.

### 3.8 `ip addr`

**Purpose:** Report network interface state, addresses and link information.

**Observed:**

```text
1: lo: <LOOPBACK,UP,LOWER_UP> ...
2: enp0s31f6: <BROADCAST,MULTICAST,UP,LOWER_UP> ... inet <redacted>/24 ...
3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> ... inet <redacted>/24 ...
4: docker0: <NO-CARRIER,...DOWN> ...
```

**Interpretation:** The loopback, Wi-Fi and Ethernet interfaces are administratively
UP and at least one interface obtained a DHCP address.

### 3.9 `systemctl --failed`

**Purpose:** List systemd units that failed during boot or while running.

**Observed:**

```text
● apport-autoreport.service loaded failed failed Process error reports when automatic reporting is enabled
● vmware.service            loaded failed failed LSB: This service starts and stops VMware services
...
2 loaded units listed.
```

**Interpretation:** Two units were reported failed. See Diagnostic Scenario 1.

### 3.10 `journalctl -b -p warning`

**Purpose:** Show warnings and errors from the current boot.

**Observed:** Multiple kernel, service and hardware-related messages were recorded,
including CPU vulnerability notices, ACPI warnings, Bluetooth firmware messages, thermal
zone reports, NetworkManager Wi-Fi link/association messages and GNOME session
messages.

**Interpretation:** The journal contains both normal warnings and potential issues; no
automatic conclusion is drawn without cross-checking. Full output:
[`evidence/journalctl-warning.txt`](evidence/journalctl-warning.txt).

## 4. Diagnostic Scenario 1: Failed systemd Service

**Symptom:** `systemctl --failed` reported:

```text
● apport-autoreport.service loaded failed failed Process error reports when automatic reporting is enabled
```

**Evidence:** `systemctl status apport-autoreport.service` and `journalctl -b -u
apport-autoreport.service` showed that the service entered the failed state with result
`exit-code`; the triggering process, `/usr/share/apport/whoopsie-upload-all`, exited with
status `2/INVALIDARGUMENT`. The service is triggered by `apport-autoreport.timer` and
`apport-autoreport.path`.

**Analysis:**
- The failure is in the apport crash-reporting upload process, not in the kernel or
  hardware.
- The unit is non-critical for normal system operation.

**Status:** The failed unit state is a **confirmed observation**. The underlying cause of
the upload process exit status was **not confirmed** and remains an open finding.

**Recommended action:** No immediate action is required to restore normal operation. The
logs should be reviewed if crash-report uploads are required; the service should not be
disabled without maintainer approval.

## 5. Diagnostic Scenario 2: Network Interface and Service Health

**Symptoms:** The following warnings were recorded in `journalctl -b -p warning`:

```text
Oct 26 ... NetworkManager: <warn> device (wlp2s0): link timed out.
Oct 26 ... NetworkManager: <warn> Activation: failed for connection '<redacted>'.
Oct 26 ... NetworkManager: <warn> Activation: (wifi) association took too long, failing activation.
```

**Evidence:**
- `ip addr` showed the Wi-Fi interface `wlp2s0` as `UP`, `LOWER_UP` with an IPv4 address.
- `nmcli device status` showed the Wi-Fi and Ethernet interfaces as connected.
- `systemctl status NetworkManager` showed `NetworkManager.service` as `active (running)`.
- NetworkManager logs also showed successful DHCP activation for the Ethernet interface.

**Analysis:**
- The Wi-Fi interface currently reports `UP` with a valid address, and the interface is
  managed by a running NetworkManager instance.
- The logged link timeout and failed association attempts were observed, but the current
  state indicates the network came back or was restored.

**Status:** The warnings are **confirmed log entries**. Whether they represent a current
fault or a transient condition was **not determined** from the available evidence.

**Recommended action:** Retry the connection if needed; prefer Ethernet for stability;
avoid persistent network changes without investigation.

## 6. Summary of findings

| Area | Finding | Confidence | Evidence |
|---|---|---|---|
| Failed service | `apport-autoreport.service` failed (`exit-code`) | Confirmed state; cause unconfirmed | `systemctl --failed`, `journalctl -u` |
| Failed service | `vmware.service` failed | Confirmed (VMware not installed) | `systemctl --failed` |
| Network | Wi-Fi link timeout / failed activation warnings | Confirmed logs; current state functional | `journalctl -b -p warning`, `ip addr`, `nmcli` |
| Memory | ~1.9 GiB available; no critical pressure | Observation | `free -h` |
| Storage | Root filesystem 34% used; no shortage | Observation | `df -h`, `lsblk` |
| Kernel/journal | Multiple warnings recorded | Observation | `journalctl -b -p warning` |

## 7. Safety and privacy statement

- Only read-only diagnostic commands were executed.
- No services were disabled and no persistent system changes were made.
- Evidence files contain no passwords, tokens, private keys or personal information;
  identifying values were redacted.

## 8. Deliverables

- Report: `docs/administration/vm-administration-report.md`
- Sanitised evidence: `docs/administration/evidence/`
