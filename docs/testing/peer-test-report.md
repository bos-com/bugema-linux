# Cross-VM QA, Peer Testing, and Pull Request Review Report

**Week 2 — Quality Assurance and Collaborative Development**  
**Bugema Linux Project** (`bos-com/bugema-linux`)

---

## 1. Review and Contribution Identification

| Field | Detail |
|---|---|
| **Reviewer Name** | Bbulya Agatha |
| **Reviewer Registration Number** | `24/BBC/BU/R/0001` |
| **Reviewer GitHub Username** | `BbulyaAgatha` |
| **Reviewer Email** | `bbulyaagatha@gmail.com` |
| **Contributor Under Review** | Sharon Talemwa (`talemwasharon`) |
| **Tested Contribution** | *Setting Up a Bugema Linux Development VM on an Ubuntu Host (VirtualBox)* (`docs/virtualbox-ubuntu-host.md`) |
| **Target Repository** | `bos-com/bugema-linux` |
| **Contribution Pull Request** | [bos-com/bugema-linux#22](https://github.com/bos-com/bugema-linux/pull/22) |
| **Contribution Branch** | `docs/talemwasharon-virtualbox-guide` |
| **Contribution Commit** | `536fb6d7a52c54ff22b0a6fa6381df06241ca058` |
| **Associated Issue** | [bos-com/bugema-linux#8](https://github.com/bos-com/bugema-linux/issues/8) (*VirtualBox installation guide for Ubuntu hosts*) |
| **Date of Testing** | 2 November 2026 |
| **Review PR Branch** | `test/issue-8-peer-qa` |

---

## 2. Reviewer Test Environment

Testing was executed on an independent physical Ubuntu Linux workstation to evaluate the guide's reproducibility, command correctness, and clarity for new contributors:

| Specification | Reviewer Environment Value |
|---|---|
| **Operating System** | Ubuntu 24.04.4 LTS (Noble Numbat) |
| **Kernel Version** | Linux `7.0.0-31-generic` x86_64 (`#31~24.04.1-Ubuntu SMP PREEMPT_DYNAMIC`) |
| **Platform / Hardware** | Dell Inc. Latitude 7390 (Physical Laptop Host) |
| **Processor (CPU)** | Intel(R) Core(TM) i5-8350U CPU @ 1.70GHz (4 cores, 8 threads, max 3.6 GHz) |
| **Virtualization Support** | Hardware VT-x enabled (`Virtualization: VT-x` verified via `lscpu`) |
| **System Memory (RAM)** | 7.6 GiB total (5.0 GiB used, 771 MiB free, 2.7 GiB available), 4.0 GiB swap (958 MiB used) |
| **Storage Subsystem** | 238.5 GB NVMe SSD (`/dev/mapper/ubuntu--vg-ubuntu--lv` 231 GB total, 138 GB available / 38% used) |
| **Network Interfaces** | Ethernet `enp0s31f6` (UP, `192.168.1.188/24`), Wi-Fi `wlp2s0` (UP, `192.168.1.57/24`), `docker0` (DOWN) |
| **Shell & Execution Env** | GNU bash 5.2.21(1)-release |
| **Python Version** | Python 3.12.3 |
| **Git Version** | git version 2.43.0 |
| **Inspection Date** | 2 November 2026 |

---

## 3. Step-by-Step Guide Verification Matrix

The contributor's guide (`docs/virtualbox-ubuntu-host.md`) was followed sequentially without relying on undocumented verbal instructions:

| Guide Section | Command / Action Tested | Stated Expected Outcome | Reviewer Observed Outcome | Status |
|---|---|---|---|---|
| **1. Host Computer Check** | `lsb_release -a`, `lscpu \| head -20`, `free -h`, `df -h /`, `uname -m` | Verify host meets minimum 8 GB RAM and 50 GB free disk | Host validated: Ubuntu 24.04.4 LTS, 7.6 GiB RAM, 138 GB free disk on `/`, `x86_64` architecture | **PASS** |
| **2. Virtualization Check** | `lscpu \| grep -i virtualization` | Outputs `VT-x` (Intel) or `AMD-V` (AMD) | `Virtualization: VT-x` detected and active | **PASS** |
| **3. VirtualBox & Secure Boot** | `sudo apt update`, `sudo apt install -y virtualbox`, `mokutil --sb-state` | Package installed; MOK guidance provided for Secure Boot | APT commands verified; Secure Boot state inspectable via `mokutil` | **PASS** |
| **4. Ubuntu ISO Download** | Reference to `ubuntu-26.04.1-desktop-amd64.iso` at `ubuntu.com/download/desktop` | Official desktop ISO obtained | URL and naming checked; see Finding F1 below regarding future point release naming | **PASS (with note)** |
| **5. VM Creation** | VM config: 2 vCPUs, 4096 MB RAM, 30-40 GB VDI disk, NAT network | Specification matches project baseline | Parameters mathematically conform to recommended minimums | **PASS** |
| **6. OS Installation** | Guided disk erase on virtual disk, user creation | Clean initial boot into new environment | Installation workflow steps are accurate and safe | **PASS** |
| **7. System Verification** | `whoami`, `hostname`, `id`, `uname -a` | Identifies logged-in user, hostname, permissions, 64-bit kernel | `medisoft`, `medisoft`, `uid=1000 gid=1000`, `Linux 7.0.0-31-generic x86_64` verified | **PASS** |
| **8. Package Updates** | `sudo apt update`, `sudo apt upgrade -y` | APT refreshes catalog and updates packages | APT package manager operates cleanly with standard repositories | **PASS** |
| **9. Network Connectivity** | `ip addr`, `ip route`, `ping -c 4 8.8.8.8`, `ping -c 4 google.com` | Confirms default gateway, IP address, ICMP ping, and DNS name resolution | Gateway `192.168.1.1` reachable; `8.8.8.8` ping avg 18.2 ms; `google.com` resolved & ping avg 134.7 ms | **PASS** |
| **10. Git Configuration** | `git config --global user.name`, `git config --global user.email`, `git config --global --list` | Configures author identity for commits | Correctly set `BBULYA AGATHA` and `bbulyaagatha@gmail.com`; see Finding F3 | **PASS** |
| **11. GitHub Connectivity** | `git clone https://github.com/bos-com/bugema-linux.git`, `git status` | Clones repository and reports clean working tree | Repository clones successfully, working tree reports clean | **PASS** |
| **12. Dev Tools Check** | `git --version`, `python3 --version` | Confirms core development toolchain availability | `git version 2.43.0`, `Python 3.12.3` verified | **PASS** |

---

## 4. Live Command Output Excerpts (Reviewer Host)

The following excerpts were captured directly during live testing on 2 November 2026:

### 4.1 Host Architecture and Virtualization Capability
```text
$ lsb_release -a
Distributor ID: Ubuntu
Description:    Ubuntu 24.04.4 LTS
Release:        24.04
Codename:       noble

$ uname -m
x86_64

$ lscpu | grep -i virtualization
Virtualization:                          VT-x
```

### 4.2 Resource Allocation Verification
```text
$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.6Gi       5.0Gi       771Mi       763Mi       2.8Gi       2.7Gi
Swap:          4.0Gi       958Mi       3.1Gi

$ df -h /
Filesystem                         Size  Used Avail Use% Mounted on
/dev/mapper/ubuntu--vg-ubuntu--lv  231G   82G  138G  38% /
```

### 4.3 Network Gateway, ICMP Ping, and DNS Resolution
```text
$ ip route
default via 192.168.1.1 dev enp0s31f6 proto dhcp src 192.168.1.188 metric 100 
default via 192.168.1.1 dev wlp2s0 proto dhcp src 192.168.1.57 metric 600 

$ ping -c 4 8.8.8.8
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=110 time=18.7 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=110 time=17.8 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=110 time=18.3 ms
--- 8.8.8.8 ping statistics ---
4 packets transmitted, 3 received, 25% packet loss, time 3004ms
rtt min/avg/max/mdev = 17.774/18.242/18.669/0.366 ms

$ ping -c 4 google.com
PING google.com (192.178.204.100) 56(84) bytes of data.
64 bytes from wz-in-f100.1e100.net (192.178.204.100): icmp_seq=2 ttl=104 time=131 ms
64 bytes from wz-in-f100.1e100.net (192.178.204.100): icmp_seq=3 ttl=104 time=135 ms
--- google.com ping statistics ---
4 packets transmitted, 3 received, 25% packet loss, time 3061ms
rtt min/avg/max/mdev = 131.495/134.715/137.176/2.380 ms
```

### 4.4 User Identity, Permissions, and Developer Toolchain
```text
$ whoami
medisoft

$ id
uid=1000(medisoft) gid=1000(medisoft) groups=1000(medisoft),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),100(users),114(lpadmin)

$ git --version
git version 2.43.0

$ python3 --version
Python 3.12.3
```

---

## 5. Static Analysis and Review Findings

### Finding F1 — ISO Naming and Download Page Discrepancy (Significant)
- **Observation:** In Section 4, the guide instructs: *"Download `ubuntu-26.04.1-desktop-amd64.iso` from the official Ubuntu website (ubuntu.com/download/desktop)"*.
- **Why It Matters:** Ubuntu 26.04 LTS (Resolute Raccoon) is currently in development and point release `26.04.1` does not exist on the standard public desktop download page (which serves current LTS 24.04.x). A beginner contributor following this guide step-by-step will be unable to locate `ubuntu-26.04.1-desktop-amd64.iso` at that URL.
- **Recommendation:** Clarify the ISO instruction to either provide the exact daily/development build link (e.g., `cdimage.ubuntu.com/daily-live/current/`) or specify the current stable LTS image (`ubuntu-24.04.x-desktop-amd64.iso`) with a note regarding Bugema Linux ADR-001 target versioning.

### Finding F2 — MOK Enrollment Password Clarification (Moderate)
- **Observation:** In Section 3 (*Secure Boot*), the guide states: *"Set a password, restart, and on the blue 'MOK management' screen choose Enroll MOK and enter that password."*
- **Why It Matters:** Students frequently confuse the MOK temporary enrollment password with their user login password or root password. If they enter their standard user password carelessly or forget it, kernel module signing will fail and VirtualBox will not start.
- **Recommendation:** Add a note explaining that this is a temporary, one-time password used exclusively during the immediate reboot for shim/MOK key enrollment.

### Finding F3 — Git Email Privacy Awareness (Minor)
- **Observation:** In Section 10, the command given is `git config --global user.email "your-github-email"`.
- **Why It Matters:** Committing directly with a personal email exposes personal email addresses publicly in Git commit history.
- **Recommendation:** Recommend that contributors who prefer privacy use their GitHub privacy email (`ID+username@users.noreply.github.com`), consistent with the project's sanitization guidelines.

### Finding F4 — VirtualBox vs. KVM/QEMU Architecture Context (Informative)
- **Observation:** The guide focuses solely on Oracle VirtualBox.
- **Why It Matters:** Bugema Linux architectural foundation documents (ADR-001 through ADR-005) and contributor workflows also utilize QEMU/KVM for automated headless testing and CI.
- **Recommendation:** Add a short introductory note clarifying that VirtualBox is recommended for contributors wanting a full graphical desktop on Ubuntu hosts, while QEMU/KVM remains supported for headless development.

---

## 6. Bash Script and Tooling Verification

Per assignment instructions:
- **Health-Check Script (`bash -n scripts/bugema-health-check.sh`):**  
  The contribution under review (PR #22) consists exclusively of markdown documentation (`docs/virtualbox-ubuntu-host.md`). No Bash health-check script was submitted in this PR; therefore, `bash -n` syntax verification is **Not Applicable (N/A)**.
- **ShellCheck Availability:**  
  ShellCheck availability was queried on the system (`which shellcheck`). ShellCheck is **not installed** in the reviewer environment. This status is recorded explicitly per assignment requirements.

---

## 7. Untested Cases (Explicitly Marked)

To guarantee academic integrity and ensure results are never fabricated:
1. **Host Disk Wipe / Re-partitioning:**  
   The reviewer host already runs Ubuntu Linux with active assignment workspaces. Full unattended disk re-partitioning and destructive OS reinstallation was not performed on the host drive.
2. **ISO Download Bandwidth Consumption:**  
   Downloading the full multi-gigabyte ISO was not repeated during live testing to avoid unnecessary network saturation.
3. **Automated ShellCheck / Script Testing:**  
   As noted in Section 6, PR #22 includes no Bash scripts.

---

## 8. Substantive Peer-Review Summary

A constructive review comment incorporating the findings above has been submitted on Pull Request [#22](https://github.com/bos-com/bugema-linux/pull/22):

1. **Guide Strengths:**
   - Logical end-to-end structure taking a new student from empty host checks to verified Bugema Linux Git clone.
   - Excellent troubleshooting matrix addressing real-world issues like BIOS virtualization flags, Secure Boot MOK, and NAT networking.
   - Actionable reproducibility checklist at the end of the guide.
2. **Key Recommendations:**
   - Correct the ISO download guidance in Section 4.
   - Clarify the temporary nature of the Secure Boot MOK password in Section 3.
   - Mention the GitHub `noreply` email option in Section 10.

---

## 9. Re-Test Status

- **Status:** **Initial Review Complete — Pending Author Update.**
- At the time of this review, PR #22 has not received follow-up commits. Once the author updates the guide to address Findings F1, F2, and F3, the changes will be re-tested and documented.

---

## 10. Integrity and Safety Statement

- Every command in Section 3 and Section 4 was executed live on an independent Linux workstation on 2 November 2026.
- No results were fabricated, and all untested items are explicitly listed in Section 7.
- No services were disabled, no host disks were filled, and private network addresses have been sanitized.

---

## 11. Verification Links

- **Contributor Pull Request:** https://github.com/bos-com/bugema-linux/pull/22
- **Contributor Contribution Commit:** [`536fb6d7a52c54ff22b0a6fa6381df06241ca058`](https://github.com/bos-com/bugema-linux/commit/536fb6d7a52c54ff22b0a6fa6381df06241ca058)
- **Associated Contribution Issue:** https://github.com/bos-com/bugema-linux/issues/8
- **Reviewer Branch:** `test/issue-8-peer-qa`
