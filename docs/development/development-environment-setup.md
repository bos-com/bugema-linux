# Bugema Linux Development Environment Setup Guide

**Project:** Bugema Linux
**Organization:** Bugema Open Source Community (BOSC), Bugema University, Uganda
**Repository:** https://github.com/bos-com/bugema-linux
**Applies to:** Bugema Linux Alpha 0.1 development
**Audience:** New contributors, students, and staff preparing to contribute to Bugema Linux

---

## 1. Purpose

Every contribution to Bugema Linux — documentation, packaging, desktop customisation,
testing, or build automation — is developed inside a Linux environment. This guide
describes how to build that environment on a personal computer and verify that it works
correctly before making a contribution.

It is the companion to:

* [`CONTRIBUTING.md`](../../CONTRIBUTING.md) — how to contribute (issue → branch → pull
  request, including the Git workflow in section 3);
* [`docs/architecture/system-architecture.md`](../architecture/system-architecture.md) — the project architecture.

The environment built here matches the architectural decisions already accepted by the
project:

| Decision | Document | Relevance to this guide |
| --- | --- | --- |
| Ubuntu 26.04 LTS base distribution | [ADR-001](../architecture/adr/ADR-001-ubuntu-2604-lts.md) | The Linux distribution installed in the virtual machine |
| Xfce 4.20 default desktop | [ADR-002](../architecture/adr/ADR-002-xfce-420-desktop.md) | Optional lightweight desktop for low-resource hosts |
| APT and `.deb` packaging | [ADR-003](../architecture/adr/ADR-003-apt-deb-package-management.md) | The package manager used throughout this guide |
| Offline-first architecture | [ADR-004](../architecture/adr/ADR-004-offline-first-architecture.md) | Why the environment is verified *before* going offline |
| Defence-in-depth security | [ADR-005](../architecture/adr/ADR-005-security-architecture.md) | Why no plaintext credential files are created |

---

## 2. Prerequisites

| Requirement | Minimum | Recommended |
| --- | --- | --- |
| Host operating system | 64-bit Windows, macOS, or Linux | 64-bit Linux |
| Processor | 4 hardware threads with VT-x or AMD-V | 8 hardware threads |
| Host memory (RAM) | 8 GB | 16 GB |
| Free disk space | 50 GB | 80 GB |
| Virtualisation support | Enabled in BIOS/UEFI | Enabled |
| Internet | Required for ISO download and package installation | — |

Check that hardware virtualisation is available on a Linux host:

```bash
grep -c -E 'vmx|svm' /proc/cpuinfo    # must print a number greater than 0
ls -l /dev/kvm                        # must exist
```

If `/dev/kvm` does not exist, enable VT-x/AMD-V in the firmware settings and install the
KVM packages for your distribution.

---

## 3. Recommended virtual machine resources

The values below are the baseline for Bugema Linux development on a typical student laptop.

| Resource | Recommended configuration | Reason |
| --- | --- | --- |
| CPU | 2 virtual CPUs | Enough for package builds and documentation work |
| RAM | 4 GB | Comfortable for the Xfce desktop plus build tools |
| Storage | 40 GB dynamic disk | Ubuntu install, build artefacts, and image builds |
| Network | NAT | Outbound Internet without exposing services to the campus network |
| Architecture | 64-bit (amd64) | Matches the project target architecture (ADR-001) |
| Display | 128 MB video memory | Sufficient for a 1080p virtual display |

> **Low-resource hosts.** If the host has 8 GB of RAM or less, install the Xfce 4.20
> desktop (Step 3.5) instead of the default desktop environment. Xfce uses considerably
> less memory and is the desktop the project has selected for Bugema Linux itself
> (ADR-002).

---

## 4. Supported virtualisation platforms

| Platform | Version tested | Notes |
| --- | --- | --- |
| Oracle VirtualBox | 7.1 or later | Widely available; used for the majority of student labs |
| VMware Workstation Pro | 17 or later | Available under the Bugema University institutional licence |
| KVM/QEMU | 8.2 or later | Linux hosts; fastest performance with KVM acceleration |
| Microsoft Hyper-V | Windows 10/11 Pro | Available where VirtualBox is not permitted |

All three platforms support the same virtual machine configuration described below. The
rest of this guide uses VirtualBox as the primary example, with equivalent instructions
for VMware and KVM/QEMU.

---

## 5. Step 1 — Obtain and verify the Ubuntu 26.04 LTS ISO

Bugema Linux is based on **Ubuntu 26.04 LTS** (ADR-001), so the development environment
must use the same distribution and version.

```bash
cd ~/Downloads
# Ubuntu 26.04.1 desktop image (official release)
wget https://releases.ubuntu.com/26.04/ubuntu-26.04.1-desktop-amd64.iso

# Verify the download against the official checksum file
wget https://releases.ubuntu.com/26.04/SHA256SUMS
sha256sum -c SHA256SUMS --ignore-missing
```

Expected output:

```text
ubuntu-26.04.1-desktop-amd64.iso: OK
```

Always verify the checksum. A corrupted or tampered ISO produces installation failures
that are very difficult to diagnose later.

---

## 6. Step 2 — Create the virtual machine

### 6.1 Oracle VirtualBox

1. Open **VirtualBox Manager → New**.
2. **Name:** `Bugema-Linux-Development`; **Type:** `Linux`; **Version:** `Ubuntu (64-bit)`.
3. **Memory size:** `4096` MB.
4. **CPUs:** `2` processors, and enable **PAE/NX** and **Nested Virtualization** under
   *System → Processor* if the host supports it.
5. **Disk:** *Create a virtual hard disk* → *VDI (Dynamic)* → `40` GB.
6. **Settings → System → Motherboard:** enable **I/O APIC** and set the boot order to
   *Optical → Hard Disk*.
7. **Settings → Display:** `128` MB of video memory, enable **3D Accelerator** if the host
   GPU supports it.
8. **Settings → Network → Adapter 1:** *Attached to:* `NAT`, *Adapter Type:* `Intel PRO/
   1000 MT Desktop (82540EM)*.
9. **Settings → USB:** select the **USB 2.0** controller for faster device enumeration.
10. **Settings → Storage:** select the empty optical drive, choose *Use a disk file* and
    point it at `ubuntu-26.04.1-desktop-amd64.iso`.
11. Click **Start**.

### 6.2 VMware Workstation Pro

1. **File → New Virtual Machine → Custom (advanced)** → *I will install the operating
   system later*.
2. Guest OS: *Linux*; Version: *Ubuntu 64-bit*.
3. **Memory:** `4096` MB; **Processors:** `2` cores.
4. **Disk:** `40` GB, *Store virtual disk as a single file*.
5. **Network:** `NAT`; **Display:** 3D acceleration enabled.
6. **CD/DVD:** *Use ISO image file* → select `ubuntu-26.04.1-desktop-amd64.iso`.
7. Click **Finish**, then power on the machine.

### 6.3 KVM/QEMU on Linux

```bash
# 40 GB qcow2 disk
qemu-img create -f qcow2 ~/vms/bugema-linux.qcow2 40G

# Boot the installer
qemu-system-x86_64 \
  -machine q35,accel=kvm \
  -cpu host \
  -smp 2 \
  -m 4096 \
  -drive file=~/vms/bugema-linux.qcow2,if=virtio,format=qcow2 \
  -cdrom ~/Downloads/ubuntu-26.04.1-desktop-amd64.iso \
  -boot d \
  -device virtio-vga \
  -netdev user,id=n1,hostfwd=tcp::2222-:22 \
  -device virtio-net-pci,netdev=n1
```

---

## 7. Step 3 — Install Ubuntu 26.04 LTS

### 7.1 Installer

1. At the VirtualBox/VMware boot screen select **Try or Install Ubuntu**, then press
   **Enter**.
2. Choose **English (UK)** or **English (Uganda)**, then **Install Ubuntu**.
3. **Keyboard layout:** English (US) by default.
4. **Network:** *Connect to a wired network* — DHCP over NAT is used automatically.
5. **Installation type:** *Erase disk and install Ubuntu*. This is safe because it targets
   the virtual disk, not the host machine. Leave the other options at their defaults.
6. **User account:** create the account that will own the work.

   | Field | Value |
   | --- | --- |
   | Your name | `NAMBALIRWA JOVIA` |
   | Computer name | `bugema-dev` |
   | Username | `jovian` |
   | Password | a strong password of your choice |
   | "Require your password to unlock" | enabled |

   A strong password is required by [ADR-005](../architecture/adr/ADR-005-security-architecture.md)
   because the environment will hold GitHub credentials.
7. **Security:** enable *Encrypt home directory* if the host disk is not encrypted.
8. **Software selection:** *Install Ubuntu desktop* (or *Minimal install* plus Xfce —
   see Step 3.5).
9. Let the installation complete, choose **Restart now**, and remove the installation ISO
   from the virtual drive when prompted.

### 7.2 First boot and login

Log in with the user created above and confirm the desktop appears. The first boot takes
longer because the system applies pending updates and initialises hardware.

### 7.3 Guest utilities

Guest utilities provide clipboard sharing, shared folders, and correct screen resizing.

* **VirtualBox:** install *Guest Additions* from the ISO that ships with your VirtualBox
  version — *Devices → Insert Guest Additions ISO* in the running guest, then run
  `sudo ./VBoxLinuxAdditions.run`. On recent Ubuntu releases the
  `virtualbox-guest-utils` package can be used instead when the host and guest versions
  match.
* **VMware:** `sudo apt update && sudo apt install -y open-vm-tools` (already installed on
  most VMware images), then `sudo systemctl restart open-vm-tools`.

Shared folders (VirtualBox) are mounted with:

```bash
sudo mkdir -p /media/sf_shared
sudo mount -t vboxsf shared /media/sf_shared
```

### 7.4 Update the system

```bash
sudo apt update
sudo apt upgrade
```

### 7.5 Optional: install the Xfce 4.20 desktop

```bash
sudo apt install -y xfce4 xfce4-goodies
```

On the next login, choose **Xfce Session** from the gear icon in the top-right corner of
the login screen. Xfce 4.20 is the desktop selected for Bugema Linux itself (ADR-002) and
is the recommended environment on low-RAM hosts.

---

## 8. Step 4 — Verify the installation

Run the following commands and confirm the output matches the descriptions.

| Command | What it proves | Example output |
| --- | --- | --- |
| `whoami` | The current user is the account you created | `jovian` |
| `id` | The user's UID, primary group, and group memberships | `uid=1000(jovian) gid=1000(jovian) groups=1000(jovian),27(sudo)` |
| `hostname` | The machine name you set during installation | `bugema-dev` |
| `hostnamectl` | Hostname, operating system, kernel, architecture, and virtualisation | `Operating System: Ubuntu 26.04.1 LTS` |
| `uname -a` | Kernel version, architecture, and build host | `Linux bugema-dev 6.x.x-...-generic #1 SMP ... x86_64 GNU/Linux` |
| `lsb_release -a` | The distribution release codename | `Distributor ID: Ubuntu`, `Release: 26.04` |
| `free -h` | Memory available to the guest | Total ~4.0 Gi |
| `df -h /` | Root filesystem size and free space | ~40 GB volume |
| `systemd-detect-virt` | Confirms the guest is virtualised | `oracle` (VirtualBox) or `kvm` |
| `ls /sys/class/dmi/id/product_name` | The virtual machine product string | `VirtualBox` |

Example session:

```console
$ whoami
jovian
$ id
uid=1000(jovian) gid=1000(jovian) groups=1000(jovian),27(sudo),100(users)
$ hostname
bugema-dev
$ uname -a
Linux bugema-dev 6.17.0-9-generic #9-Ubuntu SMP PREEMPT_DYNAMIC x86_64 GNU/Linux
$ lsb_release -a
Distributor ID:	Ubuntu
Description:	Ubuntu 26.04.1 LTS
Release:	26.04
Codename:	resolute```

---

## 9. Step 5 — Configure package management

Ubuntu uses **APT** (Advanced Package Tool) together with `dpkg` and `.deb` packages —
the package management system selected for Bugema Linux in
[ADR-003](../architecture/adr/ADR-003-apt-deb-package-management.md).

| Command | Purpose |
| --- | --- |
| `sudo apt update` | Refresh the list of available packages from the configured repositories |
| `sudo apt upgrade` | Install newer versions of installed packages |
| `sudo apt install <pkg>` | Install a package and its dependencies |
| `sudo apt remove <pkg>` | Remove a package |
| `apt list --installed \| wc -l` | Count installed packages |
| `apt-cache policy <pkg>` | Show the installed and candidate versions of a package |
| `apt-get download <pkg>` | Download a `.deb` without installing it (used for offline installs) |

```console
$ sudo apt update
Hit:1 http://archive.ubuntu.com/ubuntu noble InRelease
...
Reading package lists... Done

$ sudo apt install -y git
Setting up git (1:2.47.1-1ubuntu7) ...
```

> **Offline-first consideration.** The Bugema Linux architecture is offline-first
> (ADR-004). When working in an environment without Internet access, download the
> required `.deb` files on a connected machine and install them with
> `sudo apt install ./*.deb` or `sudo dpkg -i ./*.deb`.

---

## 10. Step 6 — Configure and test networking

```bash
# Active interfaces and their addresses
ip addr

# Default gateway and routing table
ip route

# Connectivity by IP address (no DNS involved)
ping -c 4 8.8.8.8

# Connectivity by name (tests DNS resolution as well)
ping -c 4 github.com

# Confirm how a name is resolved
getent hosts github.com

# NetworkManager connection state (Ubuntu desktop)
nmcli device status
nmcli connection show --active
```

Reading the output:

| Observation | Meaning |
| --- | --- |
| `inet 10.0.2.15/24` on `enp0s3` | The NAT address assigned by VirtualBox/VMware |
| `default via 10.0.2.2 dev enp0s3` | The NAT gateway of the virtual network |
| Replies from `8.8.8.8` | Outbound IP connectivity works |
| Replies from `github.com` | Name resolution (DNS) works as well |

If `ping 8.8.8.8` succeeds but `ping github.com` fails, the problem is name resolution,
not the network. Inspect `/etc/resolv.conf` and, on systemd-resolved systems:

```bash
resolvectl status
sudo resolvectl flush-caches
```

---

## 11. Step 7 — Install and configure Git

```bash
sudo apt update
sudo apt install -y git
git --version
```

```bash
# Identity used in every commit you make
git config --global user.name "NAMBALIRWA JOVIA"
git config --global user.email "jovianambalirwa38@gmail.com"

# Verify the configuration
git config --global --list
```

```console
$ git --version
git version 2.47.1        # example output; the exact version depends on the release
$ git config --global --list
user.name=NAMBALIRWA JOVIA
user.email=jovianambalirwa38@gmail.com
```

Use the same email address that is registered on your GitHub account, otherwise GitHub
will not link the commits to your profile.

---

## 12. Step 8 — Install GitHub CLI and authenticate

[GitHub CLI](https://cli.github.com/) (`gh`) provides authentication, pull request
management, and issue management from the terminal.

```bash
sudo apt install -y gh          # or follow https://github.com/cli/cli#installing
gh --version
```

Authenticate with the browser-based device flow:

```bash
gh auth login
```

| Prompt | Answer |
| --- | --- |
| GitHub.com | Yes |
| Preferred protocol | HTTPS |
| Authenticate Git with your GitHub credentials? | Yes |
| Login with a web browser | Copy the one-time code and approve it in the browser |

Then let `gh` manage Git credentials:

```bash
gh auth setup-git
gh auth status
```

```console
$ gh auth status
github.com
  ✓ Logged in to github.com account <your-username> (keyring)
  - Active account: true
  - Git operations protocol: https
```

> **Why not a plaintext credential file?** `git config --global credential.helper store`
> writes usernames and personal access tokens to `~/.git-credentials` **in clear text**.
> `gh auth setup-git` instead stores the token in the operating system keyring, which
> satisfies the credential-storage requirements of
> [ADR-005](../architecture/adr/ADR-005-security-architecture.md).

---

## 13. Step 9 — Clone the Bugema Linux repository and verify the workflow

```bash
# For read-only work, clone the upstream project
git clone https://github.com/bos-com/bugema-linux.git
cd bugema-linux

# For contributing, clone your own fork instead
# git clone https://github.com/<your-username>/bugema-linux.git

git status
git log --oneline -5
git branch -a
```

```console
$ git status
On branch main
Your branch is up to date with 'origin/main'.

nothing to commit, working tree clean
$ git log --oneline -5
8b13a20 docs: add Bugema Linux architecture foundation (#7)
4e7dd98 docs: add student contribution issue template (#6)
f3d702a docs: add bug report issue template (#5)
c151ed8 docs: add Bugema Linux development roadmap
b45c847 docs: add project support guide
```

The repository is now ready for the contribution workflow: create an issue, create a
branch, make the change, commit, push, and open a pull request. See
[`CONTRIBUTING.md`](../../CONTRIBUTING.md) for the complete process.

---

## 14. Development tools reference

Install the tools your work requires:

```bash
sudo apt install -y git build-essential python3 python3-pip default-jdk vim curl wget net-tools
```

| Tool | Version command | Purpose |
| --- | --- | --- |
| Git | `git --version` | Version control and the contribution workflow |
| GitHub CLI | `gh --version` | Authentication, issues, and pull requests |
| Python | `python3 --version` | Scripting, build automation, and test harnesses |
| Java | `java --version` | JVM-based tooling and the Java development stack |
| GCC / Make | `gcc --version` | Compiling native packages (ADR-003) |
| Vim | `vim --version` | Terminal text editor for documentation and code |
| curl / wget | `curl --version` | Downloading artefacts and calling the GitHub API |
| net-tools | `ifconfig -V` | Legacy network inspection (modern alternative: `ip addr`) |

---

## 15. Troubleshooting

| Problem | Cause | Solution | Result |
| --- | --- | --- | --- |
| Virtual machine does not boot | Boot order set to the empty hard disk, or the ISO was detached | *Settings → Storage* attach the ISO and set *Optical* first in the boot order | The installer starts |
| `This kernel requires x86-64` on a 64-bit host | The 32-bit ISO was downloaded | Download `ubuntu-26.04.1-desktop-amd64.iso` and verify the checksum | Correct image boots |
| `apt update` fails with `Could not resolve host` | DNS is broken in the guest | `nmcli device status`; set DNS to `8.8.8.8` for the active connection; `sudo resolvectl flush-caches` | Repository lists refresh |
| `apt update` reports `404 Not Found` | Stale package lists after a release | `sudo apt clean && sudo rm -rf /var/lib/apt/lists/* && sudo apt update` | Fresh lists restored |
| `ping` works but `git clone` hangs | Proxy or firewall blocking HTTPS (port 443) | `curl -I https://github.com` to test; check the guest proxy settings | Clone succeeds |
| `fatal: could not read Username` during `git push` | Git has no credentials for GitHub | Run `gh auth login` then `gh auth setup-git` | Push authenticated |
| `Permission denied (publickey)` | SSH key not registered or wrong key selected | Use the HTTPS remote, or add the public key with `gh ssh-key add` | Access granted |
| Guest freezes with only 2 GB RAM | Not enough memory for the desktop | Increase to 4 GB, or install the Xfce desktop (Step 3.5) | System responsive |
| `No space left on device` | 40 GB virtual disk exhausted | `sudo apt autoremove && sudo apt clean`; `df -h /` to confirm | Free space restored |
| `systemd-detect-virt` returns `none` | Nested virtualisation disabled | Enable *System → Processor → Nested Virtualization* in the platform settings | Detection works |

---

## 16. Development environment verification checklist

Before starting a contribution, confirm every item:

```text
[ ] The virtual machine boots successfully.
[ ] The student can log in.
[ ] `whoami`, `id`, `hostname` and `uname -a` return the expected values.
[ ] `sudo apt update` completes without errors.
[ ] `ip addr` shows a network interface with an address.
[ ] `ip route` shows a default gateway.
[ ] `ping -c 4 8.8.8.8` succeeds.
[ ] `ping -c 4 github.com` succeeds (DNS works).
[ ] `git --version` returns a version.
[ ] `git config --global --list` shows the correct name and email.
[ ] `gh auth status` reports an active login.
[ ] `git clone https://github.com/bos-com/bugema-linux.git` succeeds.
[ ] The required development tools are installed and runnable.
```

---

## 17. Related project documents

| Document | Purpose |
| --- | --- |
| [`README.md`](../../README.md) | Project overview, scope, and development workflow |
| [`CONTRIBUTING.md`](../../CONTRIBUTING.md) | Contribution rules, commit conventions, and review process |
| [`GOVERNANCE.md`](../../GOVERNANCE.md) | Roles, responsibilities, and decision making |
| [`SECURITY.md`](../../SECURITY.md) | How to report security issues |
| [`ROADMAP.md`](../../ROADMAP.md) | Planned milestones for Bugema Linux |
| [`docs/architecture/system-architecture.md`](../architecture/system-architecture.md) | The accepted system architecture |
| [`docs/architecture/adr/`](../architecture/adr/) | Architecture decision records |

---

## 18. References

* Ubuntu 26.04 LTS release notes and downloads — <https://releases.ubuntu.com/26.04/>
* Ubuntu documentation: installation and desktop guides — <https://docs.ubuntu.com/>
* Oracle VirtualBox manual — <https://docs.oracle.com/en/virtualbox/7.1/>
* VMware Workstation Pro documentation — <https://techdocs.broadcom.com/>
* QEMU system emulation documentation — <https://www.qemu.org/docs/master/>
* Git documentation — <https://git-scm.com/doc/>
* GitHub CLI manual — <https://cli.github.com/manual/>
* GitHub Docs: about GitHub issues and pull requests — <https://docs.github.com/en/issues>

---

**Bugema Linux**
**Bugema Open Source Community (BOSC)**
Bugema University, Uganda

> Learn. Build. Share. Innovate.
