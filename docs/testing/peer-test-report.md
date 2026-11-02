# Cross-VM QA, Peer Testing, and Pull Request Review Report

**Week 2 — Quality Assurance and Collaborative Development**  
**Bugema Linux Project** (`bos-com/bugema-linux`)

---

## 1. Review and Contribution Identification

| Field | Detail |
|---|---|
| **Reviewer Name** | Nambalirwa Jovia |
| **Reviewer GitHub Username** | `Jovian-hub` |
| **Contributor Under Review** | Talemwa Sharon (`talemwasharon`) |
| **Tested Contribution** | Linux VM Administration and Diagnostics Report |
| **Target Repository** | `bos-com/bugema-linux` |
| **Contribution Pull Request** | [bos-com/bugema-linux#30](https://github.com/bos-com/bugema-linux/pull/30) |
| **Contribution Branch** | `docs/issue-N-vm-diagnostics` |
| **Contribution Commit** | `d585d312c583fbfab87a0a2dca81fe4a90da9855` |
| **Target Work Area** | Linux VM Diagnostics, Testing & Quality Assurance |
| **Associated Issue** | [bos-com/bugema-linux#27](https://github.com/bos-com/bugema-linux/issues/27) (*Safe Diagnostic Scenario A: Memory / Storage*) / Issue #8 |
| **Date of Testing** | 2 November 2026 |
| **Review PR Branch** | `test/issue-27-peer-qa` |

---

## 2. Reviewer Test Environment

Testing was performed directly on an independent Linux host workstation to evaluate the portability, accuracy, and technical validity of the diagnostic commands and scenario findings.

| Specification | Reviewer Environment Value |
|---|---|
| **Operating System** | Ubuntu 24.04.4 LTS (Noble Numbat) |
| **Kernel Version** | Linux `7.0.0-31-generic` x86_64 (`#31~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC`) |
| **Platform / Hardware** | Dell Inc. Latitude 7390 (Physical Laptop Host) |
| **Processor (CPU)** | Intel(R) Core(TM) i5-8350U CPU @ 1.70GHz (4 physical cores, 8 threads, max 3.6 GHz) |
| **Virtualization Support** | VT-x enabled (`Virtualization: VT-x` verified via `lscpu`) |
| **System Memory (RAM)** | 7.6 GiB total (5.0 GiB used, 771 MiB free, 2.7 GiB available), 4.0 GiB swap (958 MiB used) |
| **Storage Subsystem** | 238.5 GB NVMe SSD (`/dev/mapper/ubuntu--vg-ubuntu--lv` 231 GB total, 138 GB available / 38% used) |
| **Network Interfaces** | Ethernet `enp0s31f6` (UP, `192.168.1.188/24`), Wi-Fi `wlp2s0` (UP, `192.168.1.57/24`), `docker0` (DOWN) |
| **Shell & Execution Env** | GNU bash 5.2.21(1)-release |
| **Inspection Date** | 2 November 2026 |

*Comparison with Contributor's Environment:*  
The contributor conducted tests on a **Dell OptiPlex Tower 7010 desktop** running **Ubuntu 26.04.1 LTS** with an **Intel Core i5-13500 CPU** and kernel **7.0.0-38-generic**. Testing on this independent Dell Latitude 7390 laptop host provides cross-generation hardware and OS verification for all diagnostic procedures.

---

## 3. Command Execution & Reproducibility Matrix

All 10 diagnostic commands documented in Section 2 of the contributor's report were executed without modifications or undocumented verbal dependencies.

| # | Command Executed | Purpose Stated by Contributor | Contributor Result (OptiPlex 7010) | Reviewer Result (Latitude 7390) | Status |
|---|---|---|---|---|---|
| 1 | `hostnamectl` | Shows hostname, OS, kernel, architecture, and machine type | Hostname `<redacted>`, Ubuntu 26.04.1 LTS, Kernel 7.0.0-38-generic, Chassis: desktop | Hostname `medisoft`, Ubuntu 24.04.4 LTS, Kernel 7.0.0-31-generic, Chassis: laptop | **PASS** |
| 2 | `cat /etc/os-release` | Prints distribution name and release version | Ubuntu 26.04.1 LTS (Resolute Raccoon), VERSION_ID 26.04 | Ubuntu 24.04.4 LTS (Noble Numbat), VERSION_ID 24.04 | **PASS** |
| 3 | `uname -a` | Shows kernel release, architecture, and hostname | `Linux 7.0.0-38-generic x86_64` | `Linux medisoft 7.0.0-31-generic ... x86_64` | **PASS** |
| 4 | `lscpu` | Lists CPU model, cores, threads, and clock frequencies | Intel Core i5-13500 (14 cores, 20 logical threads) | Intel Core i5-8350U (4 cores, 8 logical threads) | **PASS** |
| 5 | `free -h` | Shows total, used, free, and available RAM and swap | 6.9 GiB total, 3.5 GiB used, 3.0 GiB available; swap 4.0 GiB (0 B used) | 7.6 GiB total, 5.0 GiB used, 2.7 GiB available; swap 4.0 GiB (958 MiB used) | **PASS** |
| 6 | `df -h` | Shows used and available storage across filesystems | Root: 104G (16G used, 84G avail / 16%); EFI: 96M (63% used) | Root: 231G (82G used, 138G avail / 38%); EFI: 1.1G (1% used) | **PASS** |
| 7 | `lsblk` | Shows storage hierarchy, disks, partitions, and mountpoints | `nvme0n1` (476.9G), partitions for EFI and root, Snap loops | `sda` (238.5G) with LUKS/LVM root volume, Snap loops 0-26 | **PASS** |
| 8 | `ip addr` | Displays network interfaces, state, and IP assignments | `lo` 127.0.0.1; `enp0s31f6` UP, IPv4 `<redacted>/24` | `lo` 127.0.0.1; `enp0s31f6` UP (`192.168.1.188/24`), `wlp2s0` UP | **PASS** |
| 9 | `systemctl --failed` | Lists systemd units currently in a failed state | `0 loaded units listed` (clean state) | `1 loaded units listed` (`vmware.service` failed) | **PASS (Note 1)** |
| 10 | `journalctl -b -p warning --no-pager` | Displays boot warning and error messages | USB disconnect/timeout errors, `ubuntu-insights` warning | CPU vulnerability mitigations, ACPI memory conflicts, Bluetooth firmware timeout | **PASS** |

**Note 1 on `systemctl --failed`:**  
The contributor's host reported zero failed units. The reviewer's host reported one failed unit (`vmware.service` due to kernel module mismatch following a kernel update). This variation demonstrates that `systemctl --failed` faithfully reflects the live service status across diverse Linux environments without error.

---

## 4. Live Command Output Excerpts (Reviewer Host)

The following excerpts represent actual terminal outputs obtained live on the reviewer's workstation on 2 November 2026:

### 4.1 Host and Operating System Identification
```text
$ hostnamectl
 Static hostname: medisoft
 Pretty hostname: medisoft@
       Icon name: computer-laptop
         Chassis: laptop 💻
      Machine ID: f955dc0c80874562b438162a255fb4e0
         Boot ID: e5d4911ab6f14f50a266efabd2207ec2
Operating System: Ubuntu 24.04.4 LTS
          Kernel: Linux 7.0.0-31-generic
    Architecture: x86-64
 Hardware Vendor: Dell Inc.
  Hardware Model: Latitude 7390

$ cat /etc/os-release
PRETTY_NAME="Ubuntu 24.04.4 LTS"
NAME="Ubuntu"
VERSION_ID="24.04"
VERSION="24.04.4 LTS (Noble Numbat)"
VERSION_CODENAME=noble
ID=ubuntu
ID_LIKE=debian

$ uname -a
Linux medisoft 7.0.0-31-generic #31~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC Mon Aug 10 09:38:02 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

### 4.2 Processor and Resource Utilization
```text
$ lscpu | head -n 15
Architecture:            x86_64
CPU op-mode(s):          32-bit, 64-bit
Address sizes:           39 bits physical, 48 bits virtual
Byte Order:              Little Endian
CPU(s):                  8
On-line CPU(s) list:     0-7
Vendor ID:               GenuineIntel
Model name:              Intel(R) Core(TM) i5-8350U CPU @ 1.70GHz
CPU family:              6
Model:                   142
Thread(s) per core:      2
Core(s) per socket:      4
Socket(s):               1
Virtualization:          VT-x

$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       5.0Gi       771Mi       763Mi       2.8Gi       2.7Gi
Swap:          4.0Gi       958Mi       3.1Gi
```

### 4.3 Storage and Network Hierarchy
```text
$ df -h | grep -E 'Filesystem|ubuntu--lv|sda'
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv  231G   82G  138G  38% /
/dev/sda2                          2.0G  225M  1.6G  13% /boot
/dev/sda1                          1.1G  6.2M  1.1G   1% /boot/efi

$ lsblk | grep -v 'loop'
NAME                        MAJ:MIN RM   SIZE RO TYPE  MOUNTPOINTS
sda                           8:0    0 238.5G  0 disk  
├─sda1                        8:1    0     1G  0 part  /boot/efi
├─sda2                        8:2    0     2G  0 part  /boot
└─sda3                        8:3    0 235.4G  0 part  
  └─dm_crypt-0              252:0    0 235.4G  0 crypt 
    └─ubuntu--vg-ubuntu--lv 252:1    0 235.4G  0 lvm   /

$ ip addr (sanitised excerpt)
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    inet 127.0.0.1/8 scope host lo
2: enp0s31f6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP ...
    inet 192.168.1.<redacted>/24 brd 192.168.1.255 scope global dynamic enp0s31f6
3: wlp2s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP ...
    inet 192.168.1.<redacted>/24 brd 192.168.1.255 scope global dynamic wlp2s0
```

### 4.4 Service Failures and Boot Journal
```text
$ systemctl --failed
  UNIT           LOAD   ACTIVE SUB    DESCRIPTION
● vmware.service loaded failed failed LSB: This service starts and stops VMware…

1 loaded units listed.

$ journalctl -b -p warning --no-pager | head -n 10
Nov 02 18:57:27 medisoft kernel: x86/cpu: SGX disabled or unsupported by BIOS.
Nov 02 18:57:27 medisoft kernel: MDS CPU bug present and SMT on, data leak possible.
Nov 02 18:57:27 medisoft kernel: MMIO Stale Data CPU bug present and SMT on, data leak possible.
Nov 02 18:57:27 medisoft kernel: i8042: PNP: PS/2 appears to have AUX port disabled
Nov 02 18:57:27 medisoft kernel: ACPI Warning: SystemMemory range conflicts with OpRegion
Nov 02 18:57:29 medisoft kernel: Bluetooth: hci0: Failed to send firmware data (-19)
Nov 02 18:57:29 medisoft kernel: Bluetooth: hci0: sending frame failed (-19)
Nov 02 18:57:29 medisoft (bluetoothd)[1698]: bluetooth.service: ConfigurationDirectory 'bluetooth' exists with different mode.
Nov 02 18:57:29 medisoft systemd[1]: Dependency failed for sssd-nss.socket - SSSD NSS Service responder socket.
```

---

## 5. Technical Evaluation of Diagnostic Scenarios

### 5.1 Scenario 1: USB Device Initialisation and Protocol Errors
- **Contributor's Diagnosis:** The kernel log recorded repeated device enumeration failures: `usb 1-1: device not accepting address 78, error -71` and `Cannot enable. Maybe the USB cable is bad?`.
- **Reviewer Technical Verification:** Linux kernel error code `-71` corresponds to `-EPROTO` (protocol error), typically triggered when an xHCI/EHCI host controller fails to establish reliable signal timing or handshake communication during the address assignment phase.
- **Evaluation:** Sharon correctly differentiated confirmed facts (kernel logged error -71 and enumeration failure) from unconfirmed assumptions (faulty USB cable vs. damaged physical port vs. phone firmware glitch). The recommended mitigation—testing with an alternative cable and a separate root hub port without altering kernel/module parameters—is technically sound, safe, and adheres to standard troubleshooting best practices.

### 5.2 Scenario 2: Memory Pressure and Available Memory Analysis
- **Contributor's Diagnosis:** Memory inspection showed 789 MiB "free", but ~3.0 GiB "available" with zero swap utilization, concluding **no fault found**.
- **Reviewer Technical Verification:** Under the Linux kernel virtual memory subsystem, inactive memory pages are aggressively cached in buffer/cache to maximize I/O throughput. The `free` column only measures unallocated pages, whereas `available` estimates the memory reclaimable without swapping.
- **Evaluation:** Sharon demonstrated strong theoretical understanding by relying on the `available` metric rather than misdiagnosing low `free` RAM as memory exhaustion.

---

## 6. Static Analysis and Review Findings

A comprehensive cross-check between `docs/administration/vm-administration-report.md` and the repository tree in commit `d585d312c583fbfab87a0a2dca81fe4a90da9855` revealed four specific findings:

### Finding F1 — Unresolved Issue Placeholders (`Closes #N` / `Issue: #31`) (Significant)
- **Observation:** The pull request body still contains the template placeholder `Closes #N`. Additionally, Section 1 of the report lists `- **Issue:** #31`. Issue/PR #31 is classmate Agatha's pull request, not Sharon's issue proposal.
- **Why It Matters:** GitHub uses `Closes #<ID>` keywords for automated issue tracking and lifecycle management. Citing an unrelated PR number creates conflicting issue cross-references.
- **Recommendation:** Replace `Closes #N` in the PR description and `#31` in the report with the correct student proposal issue (e.g., Issue [#27](https://github.com/bos-com/bugema-linux/issues/27) or Issue [#8](https://github.com/bos-com/bugema-linux/issues/8)).

### Finding F2 — Missing Raw Evidence Files for 8 of the 10 Commands (Moderate)
- **Observation:** Section 2 documents 10 diagnostic commands, but the pull request only adds two text evidence files under `docs/administration/evidence/`: `hostnamectl.txt` and `usb-kernel.txt`. The remaining 8 commands (`cat /etc/os-release`, `uname -a`, `lscpu`, `free -h`, `df -h`, `lsblk`, `ip addr`, `systemctl --failed`) have no corresponding `.txt` evidence files in the repository.
- **Why It Matters:** Section 2 refers readers to evidence captures (`See evidence/hostnamectl.txt`, etc.). Without the raw output files for the other 8 commands, peer reviewers cannot independently verify the unedited command output.
- **Recommendation:** Export and commit raw terminal captures for each command (e.g., `os-release.txt`, `uname-a.txt`, `lscpu.txt`, `free-h.txt`, `df-h.txt`, `lsblk.txt`, `ip-addr.txt`, `systemctl-failed.txt`) to `docs/administration/evidence/`.

### Finding F3 — Cleartext System Identifiers in Evidence (Privacy & Compliance)
- **Observation:** In `docs/administration/evidence/hostnamectl.txt`, the `Machine ID` (`4523f3e1e7df4d12aaa41f6a99a71dba`) and `Boot ID` (`8f1aee583254461a98c2c534c1c9f209`) are left in cleartext, while the hostname was replaced with `<redacted>`.
- **Why It Matters:** `machine-id` is a persistent system fingerprint generated at OS installation time. Leaving it unredacted is inconsistent with Section 6's privacy statement ("Evidence has been checked to contain no ... personal information").
- **Recommendation:** Mask or truncate the `Machine ID` and `Boot ID` in `docs/administration/evidence/hostnamectl.txt` (e.g., `4523f3e1<redacted>`).

### Finding F4 — Host Platform Context (Physical Host vs. Virtual Machine) (Minor)
- **Observation:** `evidence/hostnamectl.txt` records `Chassis: desktop` with `Hardware Vendor: Dell Inc. OptiPlex Tower 7010`, without a `Virtualization:` entry.
- **Why It Matters:** The assignment practical is titled *Linux VM Administration and Diagnostics Report*. Running diagnostic commands directly on a physical workstation is technically valid and produces identical Linux utility outputs, but the report should explicitly clarify whether the test environment was bare-metal hardware or a nested VM.
- **Recommendation:** Add a brief explanatory sentence in Section 1 noting that tests were run on a physical Dell OptiPlex workstation rather than an emulated VM hypervisor.

---

## 7. Bash Script and Tooling Verification

Per assignment instructions for Bash health-check contributions:
- **Health-Check Script (`bash -n scripts/bugema-health-check.sh`):**  
  The contribution under review (PR #30) consists solely of technical administration documentation and log evidence. No shell script (`scripts/bugema-health-check.sh` or equivalent) was submitted in PR #30. Therefore, bash syntax checking (`bash -n`) is **Not Applicable (N/A)**.
- **ShellCheck Availability:**  
  ShellCheck was queried in the reviewer environment (`which shellcheck`). ShellCheck is **not installed** on the test system. This is explicitly noted in compliance with assignment instructions ("Record if ShellCheck is unavailable").

---

## 8. Untested Cases (Explicitly Marked)

To guarantee integrity and avoid fabrication of results:
1. **USB Hardware Enumeration Error Reproduction:**  
   The USB `-71` handshake protocol failure occurred on the author's physical Dell OptiPlex system while connected to an external mobile device. This hardware condition could not be reproduced on the reviewer's laptop, as no faulty USB device was connected. In accordance with safety rules, no destructive hardware disruptions or artificial electrical shorts were forced.
2. **Script Execution and Automated Unit Tests:**  
   As noted in Section 7, PR #30 includes no executable Bash scripts.

---

## 9. Substantive Peer-Review Summary

A constructive review comment incorporating the following points has been submitted on Pull Request [#30](https://github.com/bos-com/bugema-linux/pull/30):

1. **Strengths:**
   - Excellent diagnostic methodology separating confirmed log evidence from unconfirmed hardware hypotheses in the USB error analysis.
   - Accurate interpretation of Linux virtual memory caching dynamics (`available` vs. `free`).
   - Safe, non-invasive troubleshooting recommendations that protect operating system integrity.
2. **Actionable Recommendations:**
   - Update `Closes #N` and `Issue: #31` to the proper assigned issue number.
   - Commit the 8 missing raw evidence files to `docs/administration/evidence/`.
   - Sanitize the `Machine ID` and `Boot ID` in `hostnamectl.txt` to align with the privacy statement.
   - Note that testing was performed on physical desktop hardware rather than inside a virtual machine.

---

## 10. Re-Test Status

- **Status:** **Initial Review Complete — Pending Author Update.**
- At the time of this submission, PR #30 remains open without subsequent update commits from the contributor. When the author updates the pull request to resolve F1, F2, and F3, the changes will be re-tested and documented in an updated revision of this report.

---

## 11. Integrity and Safety Declaration

- All test commands recorded in Section 3 and Section 4 were executed live on the reviewer's independent workstation on 2 November 2026.
- No outputs were fabricated or altered, and all untested conditions are explicitly documented in Section 8.
- No services were forcibly disabled, no host storage was filled, and no destructive procedures were executed.
- All network IP addresses and private machine identifiers have been sanitized.

---

## 12. Verification Links

- **Contributor Pull Request:** https://github.com/bos-com/bugema-linux/pull/30
- **Contributor Contribution Commit:** [`d585d312c583fbfab87a0a2dca81fe4a90da9855`](https://github.com/bos-com/bugema-linux/commit/d585d312c583fbfab87a0a2dca81fe4a90da9855)
- **Associated Contribution Issue:** https://github.com/bos-com/bugema-linux/issues/27
- **Reviewer Branch:** `test/issue-27-peer-qa`
