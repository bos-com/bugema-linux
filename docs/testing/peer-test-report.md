# Peer Test Report — Issue #26 Network and Service Health Diagnostics

| Field | Value |
|---|---|
| Reviewer | Daniella Kyusa (@Daniellakyusa) |
| Contribution reviewed | Linux VM Administration and Diagnostics Report by **BbulyaAgatha** |
| Contribution issue | [#26 — Network and Service Health Diagnostics](https://github.com/bos-com/bugema-linux/issues/26) |
| Contribution pull request | [#31 — docs: add VM administration and diagnostics report](https://github.com/bos-com/bugema-linux/pull/31) |
| Contribution branch | `origin/docs/issue-26-vm-diagnostics` |
| Contribution commit | `9440091edf22918e58682070a1dddea84f4f040f` |
| Test environment | See Section 1 |
| Test date | 2 November 2026 |
| Report branch | `test/issue-26-peer-qa` |
| Report PR | [#35 — Add peer test report for issue #26](https://github.com/bos-com/bugema-linux/pull/35) |

---

## 1. Test environment

All values below were captured live on the reviewer's machine on 2 November 2026 by
running the same commands the contributor used.

| Item | Observed value | Source command |
|---|---|---|
| Distribution | Ubuntu, `ID=ubuntu`, `ID_LIKE=debian` | `cat /etc/os-release` |
| Version | **Ubuntu 26.04.1 LTS** (Resolute Raccoon), `VERSION_ID="26.04"` | `cat /etc/os-release` |
| Kernel | `6.18.54.1-microsoft-standard-WSL2`, x86_64 | `uname -a` |
| Hostname | `DESKTOP-26L97UN` | `hostnamectl` |
| Chassis / platform | container; **WSL2** on Windows; hypervisor vendor Microsoft; virtualization `wsl` | `hostnamectl`, `lscpu` |
| Architecture | x86-64 | `uname -a`, `lscpu` |
| CPU | Intel Core i3-10110U @ 2.10 GHz; 4 CPUs (2 cores, 2 threads/core), 1 socket; `VT-x` | `lscpu` |
| Memory | 7.7 GiB total, 473 MiB used, 6.7 GiB free, 7.2 GiB available; Swap 2.0 GiB (0 B used) | `free -h` |
| Storage | root on `/dev/sdd` 1007 GiB (1.6 GiB used, 955 GiB avail, 1%); swap on `/dev/sdc` 2 GiB; host drives visible at `/mnt/c` (783 G) and `/mnt/d` (148 G) | `df -h`, `lsblk` |
| Network | `lo` UP; `eth0` UP, LOWER_UP with a private IPv4 (NAT via WSL, redacted); no Wi-Fi (container) | `ip addr` |
| Failed units | 0 (none) | `systemctl --failed` |
| Test date | 2 November 2026 | — |

> **Note on platform:** The reviewer's environment is WSL2 (a container-style Linux
> subsystem on Windows), not a full traditional VM. It is a genuine Linux environment
> and is sufficient to validate that the contributor's commands run and produce valid,
> reproducible output. Specific host values (CPU model, memory, disk, interfaces)
> differ from the contributor's laptop because it is a different machine; this is
> expected and does not indicate an error in either report.

## 2. Contribution under review

BbulyaAgatha's contribution is a **documentation** deliverable (a VM administration and
diagnostics report with sanitised evidence files), not a Bash health-check script. It
lives on branch `docs/issue-26-vm-diagnostics` at commit `9440091` and is submitted via
PR #31. Files in the contribution:

- `docs/administration/vm-administration-report.md` — the report
- `docs/administration/evidence/*.txt` — evidence for: `hostnamectl`, `os-release`,
  `uname-a`, `lscpu`, `free-h`, `df-h`, `lsblk`, `ip-addr`, `systemctl-failed`,
  `journalctl-warning`, plus `apport-autoreport-service.txt` and `networkmanager-network.txt`

Because the contribution contains **no shell script**, the `bash -n` and `shellcheck`
checks (assignment task 5) are **not applicable**. ShellCheck is also not installed in
the reviewer's environment; it was not needed here. (See Section 7.)

## 3. Methodology

Peer testing of a documentation report means: re-run the same diagnostic commands on an
independent environment, confirm the commands are valid and reproducible, and
cross-check the contributor's documented claims against their own evidence files. Each
command below was executed live and its output captured directly. Results were not
fabricated; where a behaviour could not be reproduced on the reviewer's environment it
is explicitly marked untested (Section 7).

## 4. Test results

Status legend: **PASS** = command ran and produced valid, sensible output; the specific
values may differ because this is a different host. **N/A** = not applicable to this
environment.

| # | Command | Expected (per contribution) | Actual (reviewer env) | Status |
|---|---|---|---|---|
| 1 | `hostnamectl` | Static/pretty hostname, chassis, OS, kernel, arch | `DESKTOP-26L97UN`, chassis container, OS Ubuntu 26.04.1 LTS, kernel 6.18.54.1-microsoft-standard-WSL2, x86-64 | PASS |
| 2 | `cat /etc/os-release` | Dist name, version, base dist | `PRETTY_NAME="Ubuntu 26.04.1 LTS"`, `VERSION_ID="26.04"`, `ID=ubuntu`, `ID_LIKE=debian` | PASS |
| 3 | `uname -a` | Kernel name/release/version/arch | `Linux DESKTOP-26L97UN 6.18.54.1-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC ... x86_64 GNU/Linux` | PASS |
| 4 | `lscpu` | CPU arch, model, cores, freq, virt | x86_64; Intel Core i3-10110U @ 2.10GHz; 4 CPUs; `VT-x`; hypervisor Microsoft | PASS |
| 5 | `free -h` | Total/used/free/avail mem + swap | Mem 7.7Gi total / 473Mi used / 7.2Gi avail; Swap 2.0Gi (0 used) | PASS |
| 6 | `df -h` | Filesystem usage | root `/dev/sdd` 1007G (1% used); WSL mounts; host `/mnt/c` 783G (34%) | PASS |
| 7 | `lsblk` | Block device hierarchy | `sda` 722.9M, `sdb` 97M, `sdc` 2G [SWAP], `sdd` 1T (root) | PASS |
| 8 | `ip addr` | Interface state and addresses | `lo` UP; `eth0` UP, LOWER_UP (private IPv4, redacted); no Wi-Fi | PASS |
| 9 | `systemctl --failed` | List failed units | "0 loaded units listed" — no failures on reviewer's env | PASS (see note) |
| 10 | `journalctl -b -p warning` | Warnings/errors from current boot | WSL init timeout, chronyd NTS/NTP failures, apparmor sysctl warnings, TCP GRO warning | PASS |

**Note on command 9:** The reviewer's environment reported **no failed units**, whereas
the contributor's environment reported two (`apport-autoreport.service`, `vmware.service`).
This difference is environment-specific and does **not** invalidate the contributor's
recorded evidence — it confirms `systemctl --failed` is a valid, working command that
correctly reflects whatever failures exist on the host it runs on.

### 4.1 Captured output excerpts (reviewer environment)

```text
$ hostnamectl
 Static hostname: DESKTOP-26L97UN
         Chassis: container
      Virtualization: wsl
Operating System: Ubuntu 26.04.1 LTS
          Kernel: Linux 6.18.54.1-microsoft-standard-WSL2
    Architecture: x86-64

$ cat /etc/os-release
PRETTY_NAME="Ubuntu 26.04.1 LTS"
VERSION_ID="26.04"
VERSION="26.04.1 LTS (Resolute Raccoon)"
ID=ubuntu
ID_LIKE=debian

$ uname -a
Linux DESKTOP-26L97UN 6.18.54.1-microsoft-standard-WSL2 #1 SMP PREEMPT_DYNAMIC Tue Oct  6 19:13:07 UTC 2026 x86_64 GNU/Linux

$ lscpu | head
Architecture: x86_64
CPU(s): 4
Model name: Intel(R) Core(TM) i3-10110U CPU @ 2.10GHz
Virtualization: VT-x
Hypervisor vendor: Microsoft

$ free -h
               total        used        free      shared  buff/cache   available
Mem:           7.7Gi       473Mi       6.7Gi       4.1Mi       655Mi       7.2Gi
Swap:          2.0Gi          0B       2.0Gi

$ df -h | grep -E 'sdd|mnt/c'
/dev/sdd       1007G  1.6G  955G   1% /
C:\             783G  265G  518G  34% /mnt/c

$ lsblk
NAME MAJ:MIN RM   SIZE RO TYPE MOUNTPOINTS
sda    8:0    0 722.9M  1 disk
sdb    8:16   0    97M  1 disk /usr/lib/linux-tools/...
sdc    8:32   0     2G  0 disk [SWAP]
sdd    8:48   0     1T  0 disk /mnt/wslg/distro /

$ ip addr (excerpt)
1: lo: <LOOPBACK,UP,LOWER_UP> ... inet 127.0.0.1/8 ...
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> ... inet 172.24.x.x/20 ...

$ systemctl --failed
  UNIT LOAD ACTIVE SUB DESCRIPTION
0 loaded units listed.

$ journalctl -b -p warning (excerpt)
... systemd-sysctl: Couldn't write '1' to 'kernel/apparmor_restrict_unprivileged_userns', ignoring: No such file or directory
... WSL (2 - init-systemd(Ubuntu)) ERROR: /sbin/init failed to start within 10000ms
... chronyd: Can't synchronise: no selectable sources (4 unreachable sources)
... kernel: TCP: eth0: Driver has suspect GRO implementation, TCP performance may be compromised.
```

## 5. Static cross-check findings (report vs. its own evidence)

The contributor's report was cross-checked against the evidence files in the same
commit.

**Finding F1 — Kernel string is inconsistent with the documented OS (significant).**
The report (Section 2 and Section 3.3) and the evidence files
`evidence/uname-a.txt` and `evidence/hostnamectl.txt` all state the kernel as
`7.0.0-31-generic` while the OS is recorded as **Ubuntu 24.04.4 LTS**. Ubuntu 24.04.x
ships the **6.8.x** kernel series; no `7.0.0` kernel exists in the standard 24.04 or
HWE stack. The reviewer could not reconcile this value from the provided evidence.
*Why it matters:* the kernel string is a primary system-identification artifact; an
implausible value undermines the reliability of the report and could mislead anyone
relying on it. *Recommendation:* re-verify with `uname -r` on the VM and correct the
value, or add a note explaining the use of a custom/locally-built kernel if one is
genuinely in use.

**Finding F2 — Machine ID / Boot ID left in cleartext (minor, privacy).**
In `evidence/hostnamectl.txt` the `Machine ID` (`f955dc0c…`) and `Boot ID`
(`0108960fca…`) are left in cleartext, while the hostname and IP addresses were
redacted. `machine-id` is a persistent unique host identifier. The report's own safety
statement (Section 7) claims "identifying values were redacted." *Why it matters:* it is
a small inconsistency with the stated redaction practice. *Recommendation:* redact or
truncate these IDs for consistency with the privacy statement.

## 6. Substantive peer-review observations

These observations are also posted as a review comment on PR #31.

1. **Strength — clean, reproducible methodology.** Every command has a Purpose /
   Observed / Interpretation structure with a link to its evidence file. All 10
   commands were re-run on an independent environment (Ubuntu 26.04.1 LTS / WSL2,
   Intel i3-10110U) and produced valid output, confirming the methodology is
   environment-agnostic. (Reviewer verified, Section 4.)
2. **Strength — disciplined status language.** The two diagnostic scenarios
   distinguish "confirmed observation" from "unconfirmed cause" rather than
   overclaiming. This is good practice.
3. **Strength — self-flagged OS discrepancy.** The report notes that it observed
   24.04.4 LTS where ADR-001 targets 26.04 LTS. (For reference, the reviewer's
   environment is 26.04.1 LTS, so the same commands work on the target version too.)
4. **Issue — kernel value (see F1).** The `7.0.0-31-generic` kernel is implausible for
   Ubuntu 24.04.4 LTS and should be verified/corrected. This is the most important
   finding.
5. **Issue — incomplete redaction (see F2).** Machine ID and Boot ID remain in
   `hostnamectl.txt`; redact for consistency with the privacy statement.
6. **Minor — `vmware.service` failure.** The summary records `vmware.service` as failed
   with "VMware not installed." A one-line note on why a service for an uninstalled
   product is present (typically a leftover package) would help future readers.

## 7. Untested cases (explicitly marked)

- **Diagnostic scenarios not reproduced.** The contributor's two scenarios (failed
  `apport-autoreport.service`/`vmware.service`, and Wi-Fi link timeouts) are
  host-specific. The reviewer's WSL2 container reported **0 failed units** and has **no
  Wi-Fi interface**, so these specific symptoms could not be reproduced here. They are
  accepted as the contributor's recorded evidence; only the underlying commands
  (`systemctl --failed`, `journalctl -b -p warning`, `ip addr`) were verified to run
  validly.
- **No script → no `bash -n` / `shellcheck`.** The contribution contains no shell
  script, so syntax/lint checks are not applicable. ShellCheck is not installed in the
  reviewer's environment and was not required.
- **No failure-scenario forcing.** Per the assignment's safety guidance, no disk was
  filled and no essential service was disabled to force a failure. This was intentional.

## 8. Re-test status

The contributor's PR #31 is still open and this review is being submitted now. No fixes
have been applied yet, so there is nothing to re-test at this time. If the author
addresses F1/F2, the reviewer will re-verify and record the outcome here.

## 9. Integrity statement

- Every result in Section 4 was produced by running the listed commands live in the
  reviewer's environment on 2 November 2026 and capturing the output directly.
- No test result was fabricated. Cases that could not be tested are explicitly marked
  in Section 7.
- Command values differ from the contributor's because a different host was used, which
  is expected and does not indicate an error in either report.
- Evidence contains no passwords, tokens, private keys, or personal information; the
  reviewer's private IP is redacted.

## 10. Links

- Contribution issue: https://github.com/bos-com/bugema-linux/issues/26
- Contribution pull request: https://github.com/bos-com/bugema-linux/pull/31
- Contribution commit: `9440091edf22918e58682070a1dddea84f4f040f`
- This report's pull request: https://github.com/bos-com/bugema-linux/pull/35
