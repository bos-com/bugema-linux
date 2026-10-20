# Bugema Linux Alpha 0.1 — Image Verification and Smoke Test Guide

**Project:** Bugema Linux
**Organization:** Bugema Open Source Community (BOSC), Bugema University, Uganda
**Repository:** https://github.com/bos-com/bugema-linux
**Applies to:** Bugema Linux Alpha 0.1
**Audience:** Contributors, testers, students, and staff who test a Bugema Linux image

---

## 1. Purpose

Every Bugema Linux release will be distributed as an installable image, and every image
has to be **verified** before it is trusted: verified before it is written to a USB
stick, and verified again after it has been installed into a virtual machine.

This guide defines that verification. It covers:

1. obtaining a release image and proving that it has not been corrupted or tampered
   with (checksum verification);
2. writing the image to removable media correctly;
3. booting the image in a virtual machine with a known configuration;
4. running a **smoke test** — a short, repeatable set of checks that establish whether
   the image is fit to be used at all;
5. recording the result in a standard form, so that results from different machines
   can be compared; and
6. documenting a failure as a known issue, in the form the project can act on.

It is the companion to:

* [`CONTRIBUTING.md`](../../CONTRIBUTING.md) — the contribution workflow, and the
  instructions for building the development environment in which the tests are run
  (see also the developer environment guide currently in review);
* [`ROADMAP.md`](../../ROADMAP.md) — which lists *ISO checksum*, *known-issues
  documentation* and *test ISO in VirtualBox* among the Alpha 0.1 acceptance criteria;
* [`SECURITY.md`](../../SECURITY.md) — how to report a problem with a release;
* [`docs/architecture/system-architecture.md`](../architecture/system-architecture.md)
  — what the image is expected to contain.

---

## 2. Why a verification procedure is needed

A distribution image is a large binary artefact that is copied between machines,
written to USB media, and installed. Three things can go wrong, and only one of them is
obvious:

| Failure | Symptom | How this guide prevents it |
| --- | --- | --- |
| **Corrupted transfer** | The installer crashes, or the file hash does not match | Checksum verification before use (Section 4) |
| **Tampered artefact** | The image behaves subtly differently from the official build | Checksum verification against a checksum file published with the release (Section 4) |
| **Incomplete or broken build** | The image boots but the network, the package manager, Git, or a core service is not working | The smoke test in Section 6, executed on every image before it is called a release |

The third failure is the dangerous one, because a broken image that merely fails to boot
is noticed immediately, while an image whose networking or package manager is broken is
only noticed by the first user who tries to install software on it. A repeatable smoke
test turns "something is wrong" into a specific, reportable failure.

This procedure is also what makes the project's *offline-first* commitment
(ADR-004) testable: if the image passes its smoke test on a machine that has never been
connected to the Internet, it can be relied upon in a laboratory with no connectivity.

---

## 3. What the image is expected to contain

The smoke test below checks the image against the architecture the project has
accepted, so that a failure points at a real requirement rather than at a personal
preference:

| Requirement | Source | What the image must show |
| --- | --- | --- |
| Ubuntu 26.04 LTS base distribution | [ADR-001](../architecture/adr/ADR-001-ubuntu-2604-lts.md) | `lsb_release -a` reports release `26.04` |
| Xfce 4.20 desktop | [ADR-002](../architecture/adr/ADR-002-xfce-420-desktop.md) | A graphical session is present and starts |
| APT and `.deb` package management | [ADR-003](../architecture/adr/ADR-003-apt-deb-package-management.md) | `apt --version` and `dpkg --version` work, and `apt update` succeeds |
| Offline-first operation | [ADR-004](../architecture/adr/ADR-004-offline-first-architecture.md) | The system is usable with no network, and the package lists can be refreshed when a network is available |
| Defence-in-depth security | [ADR-005](../architecture/adr/ADR-005-security-architecture.md) | A non-default user account exists with a password, and no service is listening that should not be |

---

## 4. Step 1 — Obtain the image and verify the checksum

Download the image and the published checksum file **from the same release
directory**, so that the checksum always corresponds to the exact file that was
published:

```console
$ cd ~/Downloads
$ wget https://releases.ubuntu.com/26.04/ubuntu-26.04.1-desktop-amd64.iso
$ wget https://releases.ubuntu.com/26.04/SHA256SUMS
```

Verify the image:

```console
$ sha256sum -c SHA256SUMS --ignore-missing
ubuntu-26.04.1-desktop-amd64.iso: OK
```

Record the checksum itself, because it is the identifier of the build:

```console
$ sha256sum ubuntu-26.04.1-desktop-amd64.iso
1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b  ubuntu-26.04.1-desktop-amd64.iso
```

> **A checksum that does not match must be treated as a failed image.** Do not retry
> the installation and do not report the mismatch as a "flaky" failure: re-download the
> file, and if the mismatch persists, report it as a release problem using the process
> in Section 8.

**Verifying on Windows.** Use PowerShell:

```powershell
Get-FileHash -Algorithm SHA256 .\ubuntu-26.04.1-desktop-amd64.iso
```

**Verifying on macOS.** The command is the same as on Linux:

```console
$ shasum -a 256 -c SHA256SUMS
```

---

## 5. Step 2 — Write the image to removable media

Identify the device first. On Linux:

```console
$ lsblk -d -o NAME,SIZE,MODEL,TRAN
NAME  SIZE  MODEL                     TRAN
sda   14.9G  SanDisk Ultra USB 3.20   usb
sdb  931.5G  Samsung SSD 970 EVO      nvme
```

Write the image, and wait for the write to be flushed before removing the device:

```console
$ sudo dd if=ubuntu-26.04.1-desktop-amd64.iso of=/dev/sda bs=4M status=progress conv=fsync
$ sync
```

> **`/dev/sdX` must be the USB device.** Writing an image to the wrong device destroys
> the data on it. Always confirm the model and size with `lsblk` before running `dd`,
> and never type a disk name that has not just been read from that output.

---

## 6. Step 3 — Boot the image and run the smoke test

### 6.1 Boot the image in a virtual machine

The reference configuration for testing is the one recommended for development, so that
a failure in the image is not confused with an under-resourced machine:

| Resource | Value |
| --- | --- |
| Machine name | `Bugema-Linux-Alpha-Test` |
| CPU | 2 virtual CPUs |
| RAM | 4096 MB |
| Disk | 40 GB |
| Network | NAT, with Internet enabled for the first run |
| Architecture | 64-bit (amd64) |
| Firmware | UEFI (Secure Boot disabled) |

Boot the installer, complete the installation, and then start the installed system for
the smoke test. On a laptop with limited resources, 2 CPUs and 4 GB of memory is
comfortable for the Xfce desktop; if the installer itself struggles, raise the memory
temporarily to 6 GB and return to 4 GB for the test.

### 6.2 The smoke test

Run the following checks **in order** and record the result of each one. A failure in
an early check invalidates the checks after it, so stop and report at the first
failure.

**A. System identity**

```console
$ lsb_release -a
Distributor ID:	Ubuntu
Description:	Ubuntu 26.04.1 LTS
Release:	26.04

$ uname -a
Linux bugema-alpha 6.17.0-9-generic #9-Ubuntu SMP PREEMPT_DYNAMIC x86_64 GNU/Linux
```

*Expected:* Ubuntu, release 26.04, `x86_64` architecture, and a kernel matching the
release (ADR-001).

**B. Resources and storage**

```console
$ nproc
$ free -h
$ df -hT /
$ lsblk
```

*Expected:* at least 2 CPUs visible, memory detected, the root filesystem present and
large enough to install packages, and the virtual disk detected as a block device.

**C. Package management**

```console
$ apt --version
$ dpkg --version
$ sudo apt update
$ dpkg -l | grep -c '^ii'
```

*Expected:* `apt` and `dpkg` report versions, `apt update` refreshes the package lists
without an error, and a plausible number of packages is installed (ADR-003).

**D. Network and name resolution**

```console
$ ip -brief addr
$ ip route
$ ping -c 4 8.8.8.8
$ ping -c 4 github.com
$ getent hosts github.com
```

*Expected:* a network interface with an address, a default gateway, replies from
`8.8.8.8` (route working) **and** from `github.com` (DNS working). If the first ping
succeeds and the second fails, record it as a **name-resolution** failure, not a
network failure.

**E. Services and logs**

```console
$ systemctl --failed --no-pager
$ systemctl is-system-running
```

*Expected:* no failed units, and the system reports `running` or `degraded` with the
degradation explained.

**F. Development tools required by the Alpha 0.1 milestone**

```console
$ git --version
$ python3 --version
$ java --version
$ gh --version
```

*Expected:* each command prints a version. Only the tools actually present in the image
should be reported, so a tool that is absent is itself a finding.

**G. Offline behaviour (ADR-004)**

```console
$ sudo systemctl stop systemd-resolved            # simulate the loss of DNS
$ ping -c 2 8.8.8.8
$ sudo systemctl start systemd-resolved
```

*Expected:* with DNS stopped, the system remains usable — the desktop starts, the file
system is read/write, and the package manager can still install from a locally cached
`.deb`. The test is that the machine does not degrade into an unusable state when the
network disappears.

**H. First-boot and graphical session**

```console
$ systemctl get-default
$ echo $XDG_CURRENT_DESKTOP
```

*Expected:* the default target is `graphical`, and the desktop environment reported is
Xfce (ADR-002).

### 6.3 Recording the result

Copy this table into the test report and fill in the **Result** column with `PASS`,
`FAIL`, or `N/A`:

| # | Check | Command | Expected | Result |
| --- | --- | --- | --- | --- |
| A1 | Distribution and release | `lsb_release -a` | Ubuntu, release 26.04 | |
| A2 | Kernel and architecture | `uname -a` | x86_64, release-matched kernel | |
| B1 | CPU count | `nproc` | ≥ 2 | |
| B2 | Memory | `free -h` | memory detected | |
| B3 | Root filesystem | `df -hT /` | filesystem present, sufficient size | |
| B4 | Block devices | `lsblk` | virtual disk detected | |
| C1 | Package manager | `apt --version`, `dpkg --version` | both report versions | |
| C2 | Package lists | `sudo apt update` | completes without error | |
| C3 | Installed packages | `dpkg -l \| grep -c '^ii'` | plausible count | |
| D1 | Interfaces | `ip -brief addr` | interface with an address | |
| D2 | Gateway | `ip route` | default route present | |
| D3 | IP connectivity | `ping -c 4 8.8.8.8` | replies received | |
| D4 | DNS | `ping -c 4 github.com` | replies received | |
| D5 | Resolution | `getent hosts github.com` | address returned | |
| E1 | Failed units | `systemctl --failed` | none | |
| E2 | System state | `systemctl is-system-running` | running / degraded | |
| F1 | Git | `git --version` | version printed | |
| F2 | Python | `python3 --version` | version printed | |
| F3 | Java | `java --version` | version printed | |
| F4 | GitHub CLI | `gh --version` | version printed | |
| G1 | Offline behaviour | DNS stopped | system still usable | |
| H1 | Default target | `systemctl get-default` | graphical | |
| H2 | Desktop | `echo $XDG_CURRENT_DESKTOP` | Xfce | |

Always record the **build identifier** with the results, because a result without it
cannot be compared with any other result:

```text
Image:        ubuntu-26.04.1-desktop-amd64.iso
SHA256:       1a2b3c4d5e6f7a8b9c0d1e2f3a4b5c6d7e8f9a0b1c2d3e4f5a6b7c8d9e0f1a2b
Host:         <hostname>, <distribution and version>
Platform:     <VirtualBox version | VMware version>
VM:           2 CPU / 4096 MB RAM / 40 GB disk / NAT
Tested on:    <YYYY-MM-DD>
Tester:       <name>
Result:       <PASS | FAIL - see notes>
```

---

## 7. Comparing a new build with the previous one

A smoke test that is run once proves very little. Run it for **every** candidate
build, and keep the results side by side, so that a regression is visible:

| Check | Build 0.1-rc1 | Build 0.1-rc2 | Changed? |
| --- | --- | --- | --- |
| SHA256 of image | `1a2b…` | `9f8e…` | expected |
| A1 distribution / release | PASS | PASS | no |
| C2 `apt update` | PASS | FAIL | **regression** |
| D4 DNS | PASS | PASS | no |
| F2 Python | PASS | PASS | no |

A check that changes from `PASS` to `FAIL` between two builds is a regression and
should be reported as a bug before the new build is released, however small the change
is.

---

## 8. Documenting a failure as a known issue

When a check fails, capture the evidence **before** changing anything on the machine.
A known issue should be documented in the following form, which is the form the
project's bug report template expects:

```text
Title:       [BUG] <one-line description of the failed check>
Severity:    <blocker | major | minor>
Build:       <image name and SHA256>
Environment: <host OS and version>, <virtualisation platform and version>,
             <2 CPU / 4096 MB RAM / 40 GB disk / NAT>
Check:       <the check identifier and command from Section 6.2>
Expected:    <what should have happened>
Actual:      <what happened, with the full command output>
Reproduction:
  1. <step>
  2. <step>
  3. <step>
Workaround:  <how the problem was worked around, if any>
```

Then open an issue on the project tracker
(<https://github.com/bos-com/bugema-linux/issues>) using the **Bug report** template,
and — if the problem is caused by the image rather than by the test environment — link
it to the release it was found in.

Two rules keep the report useful:

* **Quote the output, do not describe it.** "The network did not work" is unusable;
  the four lines of `ping` output and the output of `ip route` are exactly what a
  maintainer needs.
* **State the build.** A failure without the image checksum cannot be reproduced,
  because the image cannot be identified.

If the same failure is found on more than one machine, say so in the issue and include
both results: a failure that only happens on one machine is usually an environment
problem, not an image problem.

---

## 9. Reporting a problem with a release

If a suspected security problem is found in an image, do **not** open a public issue.
Follow the security reporting procedure in [`SECURITY.md`](../../SECURITY.md) instead,
which keeps the disclosure private until a fix is available.

Everything else — a failed check, an installation problem, a missing tool, a
documentation error in this guide — is a normal issue and should be opened in the
tracker.

---

## 10. Contributing improvements to this guide

This guide is itself a contribution, and it improves through the same workflow as the
rest of the project:

1. Check the [issue tracker](https://github.com/bos-com/bugema-linux/issues) for an
   existing issue, and open one if there is none.
2. Create a branch: `git checkout -b docs/<your-name>-<purpose>`.
3. Make the change, and record the result of any new check you add, so that the guide
   contains evidence rather than assumptions.
4. Commit with a meaningful message: `docs: add <what you changed>`.
5. Push the branch and open a pull request that references the issue with
   `Closes #<issue-number>`.
6. Respond to review comments by committing the revision to the same branch and
   pushing again.

The full process is described in [`CONTRIBUTING.md`](../../CONTRIBUTING.md).

---

## 11. References

* Bugema Linux repository and issue tracker — <https://github.com/bos-com/bugema-linux>
* Bugema Linux roadmap, including the Alpha 0.1 acceptance criteria —
  <https://github.com/bos-com/bugema-linux/blob/main/ROADMAP.md>
* Ubuntu 26.04 LTS downloads and checksum file — <https://releases.ubuntu.com/26.04/>
* Ubuntu documentation — <https://docs.ubuntu.com/>
* `sha256sum` manual page — <https://www.gnu.org/software/coreutils/>
* GNU `dd` documentation — <https://www.gnu.org/software/dd/manual/>
* `systemd` service and unit documentation — <https://www.freedesktop.org/software/systemd/>
* Oracle VirtualBox manual — <https://docs.oracle.com/en/virtualbox/7.1/>
* GitHub Docs: about issue forms and templates —
  <https://docs.github.com/en/issues>

---

**Bugema Linux**
**Bugema Open Source Community (BOSC)**
Bugema University, Uganda

> Learn. Build. Share. Innovate.
