# Setting Up a Bugema Linux Development VM on an Ubuntu Host (VirtualBox)

This guide explains how to create a Linux virtual machine for Bugema Linux development when your computer already runs Ubuntu. It follows the recommended configuration from the Bugema Linux practical handout.

**Who this is for:** students and contributors who have never configured a Linux VM.
**Result:** a working Ubuntu VM that has internet access, Git configured, and a local clone of the Bugema Linux repository.

---

## 1. Check the host computer

Run these commands in a terminal on the host and record the output:

```bash
lsb_release -a      # host operating system
lscpu | head -20    # processor and architecture
free -h             # RAM
df -h /             # storage
uname -m            # should print x86_64 (64-bit)
```

Recommended minimum host: 8 GB RAM and at least 50 GB of free disk space, so the VM can have 4 GB RAM and 30-40 GB disk.

## 2. Confirm hardware virtualization is enabled

```bash
lscpu | grep -i virtualization
```

You should see `VT-x` (Intel) or `AMD-V` (AMD). If nothing is shown, restart the computer, open the BIOS/UEFI settings, and enable **Intel Virtualization Technology** or **SVM Mode**.

## 3. Install VirtualBox

```bash
sudo apt update
sudo apt install -y virtualbox
VBoxManage --version
```

Note the version number for your report.

**Secure Boot:** if Secure Boot is enabled, the installer may ask you to create a password for a Machine Owner Key (MOK). Set a password, restart, and on the blue "MOK management" screen choose **Enroll MOK** and enter that password. Without this step the VirtualBox kernel modules will not load. Check the Secure Boot state with:

```bash
mokutil --sb-state
```

## 4. Download the Ubuntu ISO

Download `ubuntu-26.04.1-desktop-amd64.iso` from the official Ubuntu website (ubuntu.com/download/desktop) and keep it somewhere easy to find, for example `~/Downloads`.

## 5. Create the virtual machine

Open VirtualBox and click **New**. Use these settings:

| Setting | Value |
| --- | --- |
| Name | Bugema-Linux-Development |
| ISO image | ubuntu-26.04.1-desktop-amd64.iso |
| Type / Version | Linux / Ubuntu (64-bit) |
| CPU | 2 virtual CPUs |
| RAM | 4096 MB |
| Disk | 30-40 GB, dynamically allocated (VDI) |
| Network | NAT (default) |

Tip: tick **Skip Unattended Installation** if you want to go through the installer yourself and document each stage.

## 6. Install Ubuntu in the VM

1. Start the VM. It boots from the attached ISO.
2. Choose **Install Ubuntu**, then select your language and keyboard layout.
3. Choose **Erase disk and install Ubuntu**. This only affects the virtual disk, not your host.
4. Create your user: enter your name, computer name, username and a password.
5. Wait for the installation to finish, then restart. If asked, remove the installation medium and press Enter.
6. Log in with the account you created. This is the first boot.

## 7. Verify the Linux system

Open a terminal inside the VM:

```bash
whoami       # the logged-in username
hostname     # the computer name of the VM
id           # user ID, group ID and group memberships
uname -a     # kernel version and architecture
```

These commands confirm who you are logged in as, which machine you are on, your permissions, and that the system is a 64-bit Linux kernel.

## 8. Update the package system

```bash
sudo apt update     # refreshes the list of available packages
sudo apt upgrade -y # installs newer versions of installed packages
```

Ubuntu uses the **APT** package manager. `update` only downloads package information; `upgrade` actually installs updates.

## 9. Test network connectivity

```bash
ip addr                 # shows the VM's IP address (usually 10.0.2.15 on NAT)
ip route                # shows the default gateway
ping -c 4 8.8.8.8       # tests raw internet connectivity
ping -c 4 google.com    # tests internet plus DNS name resolution
```

- If `ping 8.8.8.8` works but `ping google.com` fails, the problem is DNS.
- If both fail, check that the VM's network adapter is set to NAT and that the host itself is online.

## 10. Install and configure Git

```bash
sudo apt install -y git
git --version
git config --global user.name "Your Full Name"
git config --global user.email "your-github-email"
git config --global --list
```

## 11. Test GitHub connectivity

```bash
git clone https://github.com/bos-com/bugema-linux.git
cd bugema-linux
git status
```

A successful clone and a clean `git status` show that the VM can reach GitHub and is ready for development.

## 12. Check development tools

```bash
git --version
python3 --version
```

| Tool | Purpose |
| --- | --- |
| Git | Version control |
| Python 3 | Development and scripting |

## Troubleshooting

| Problem | Likely cause | Fix |
| --- | --- | --- |
| VirtualBox only offers 32-bit systems | Virtualization disabled | Enable VT-x / AMD-V in the BIOS (section 2) |
| "Kernel driver not installed" or modules fail to load | Secure Boot blocking modules | Enroll the MOK (section 3) |
| VM is very slow | Too little RAM or CPU | Increase RAM/CPU if the host allows |
| No internet in the VM | Wrong network mode | Set adapter to NAT, restart the VM |
| `ping` works for IP but not names | DNS problem | Restart networking or the VM, then retest |

## Reproducibility checklist

- [ ] Host details recorded
- [ ] VirtualBox installed and version noted
- [ ] VM created with the recommended resources
- [ ] Ubuntu installed and first login completed
- [ ] Verification, update and network tests completed
- [ ] Git installed and configured
- [ ] Bugema Linux repository cloned successfully
