# Day 01 — Kickoff, Ethics & Linux Fundamentals

**Date:** Jul 27, 2026
**Week:** Week 1 — Foundations

---

## 🎯 Overview

Program kickoff. Before touching a single tool, we signed an ethics/code-of-conduct
agreement — everything taught from this day forward (Nmap, Burp, Mimikatz, malware
loaders...) is dual-use, and the only thing separating "pentester" from "criminal" is
written authorization. After that, the day moved into lab setup and Linux fundamentals,
since almost every offensive/defensive tool this month runs from a Linux terminal.

## 📜 Ethics & Rules of Engagement

- **Authorization first** — never run a scan, exploit, or tool against a system you don't
  own or don't have explicit written permission to test.
- **Scope matters** — an engagement's Rules of Engagement (RoE) define exactly what's
  in-scope (IP ranges, domains, time windows) and what's off-limits.
- **Confidentiality** — findings from a real engagement are client-confidential until
  disclosed through the agreed process.
- Everything in this lab environment is **host-only / air-gapped** — nothing here is aimed
  at the internet.

## 🖥️ Hypervisor & Lab Network Setup

We use a Type-2 hypervisor (VirtualBox or VMware Workstation) to run our attack and target
VMs. The critical setting is the **network adapter mode**:

| Mode | What it does | Use it for |
|------|---------------|------------|
| NAT | VM gets internet via host, but host/other VMs can't reach it | Downloading updates only |
| Bridged | VM appears as its own device on the physical LAN | Never, for this lab |
| **Host-only** | VMs can talk to each other and the host, **no path to the internet** | Our entire lab range |
| Internal Network | VMs can talk to each other only, not even the host | Fully isolated attack ranges |

**Why host-only matters:** an accidental scan or a live malware detonation (Week 3) must
never be able to reach the real internet. Host-only networking is what makes that
guarantee — it's the single most important setting from today that protects every later
lab.

## 🐧 The Linux Philosophy

- **Linux is a kernel**, not an OS by itself. A "Linux distro" = kernel + GNU userland
  tools + package manager + (optionally) a desktop environment. Kali and Parrot are
  Debian-based distros pre-loaded with security tooling.
- **Golden rule: "everything is a file."** Regular files, directories, hardware devices
  (`/dev/sda`), running processes (`/proc/<pid>/`), and even network sockets are all
  represented and manipulated through the filesystem.

### Filesystem Hierarchy

| Path | Contains | Security relevance |
|------|----------|---------------------|
| `/` | Root of the entire tree | — |
| `/bin`, `/usr/bin` | Everyday user binaries (`ls`, `cat`, `cp`) | — |
| `/sbin` | Admin/system binaries (`reboot`, `ip`) | Often needs root |
| `/etc` | System-wide config files | 🔵 Blue: watch for unauthorized edits |
| `/etc/passwd` | User account list (readable by all) | 🔴 Red: usernames for brute-force |
| `/etc/shadow` | Hashed passwords (root-only) | 🔵 Blue: must stay protected |
| `/home` | Per-user home directories | 🔴 Red: user data / SSH keys live here |
| `/root` | root's home directory | 🔴 Red: reaching this = full compromise |
| `/tmp` | World-writable scratch space, cleared on reboot | 🔴 Red: classic spot to drop a payload |
| `/var/log` | System & application logs | 🔵 Blue: primary intrusion-detection source |
| `/proc` | Live kernel/process info (virtual filesystem) | Useful for both recon and forensics |

## 🔑 File Permissions

Every file has three permission triads — **owner / group / other** — each with
**read (r) / write (w) / execute (x)**:

```
-rwxr-xr--  1 maty  security  1024 Jul 27 09:00 script.sh
 │└┬┘└┬┘└┬┘
 │ │  │   └── other: r-- (read only)
 │ │  └────── group: r-x (read + execute)
 │ └───────── owner: rwx (read + write + execute)
 └─────────── file type (- = regular file, d = directory, l = symlink)
```

Numeric (octal) form: r=4, w=2, x=1, summed per triad.
`chmod 750 script.sh` → owner=rwx(7), group=r-x(5), other=none(0).

```bash
chmod 750 script.sh        # set exact permissions
chmod u+x script.sh        # add execute for owner only
chown maty:security file   # change owner:group
```

**SUID/SGID bit** — when set on an executable, it runs with the *file owner's* privileges
instead of the caller's. A root-owned SUID binary is one of the most common
privilege-escalation vectors on Linux (this comes back in detail on **Day 9**):

```bash
ls -la /usr/bin/passwd
# -rwsr-xr-x 1 root root ...   <- 's' instead of 'x' = SUID bit set
```

## 🔐 SSH — Remote Access

SSH (Secure Shell) is how we'll reach almost every remote lab box this month.

```bash
ssh-keygen -t ed25519 -C "maty-ctc-lab"   # generate a keypair
ssh-copy-id user@target                   # push the public key to a remote host
ssh -i ~/.ssh/id_ed25519 user@target       # connect using that key
ssh -p 2222 user@target                    # connect on a non-default port
```

Key-based auth is preferred over passwords because the private key never leaves your
machine, and it can't be brute-forced the way a weak password can.

## 🎮 OverTheWire — Bandit (first steps)

Bandit is a wargame for building terminal muscle memory: each level's password unlocks the
SSH login for the next level.

```bash
ssh bandit0@bandit.labs.overthewire.org -p 2220
```

| Level | Concept | Command |
|-------|---------|---------|
| 0 → 1 | Reading a file | `cat readme` |
| 1 → 2 | Filename starting with `-` | `cat ./-` |
| 2 → 3 | Filename with spaces | `cat "spaces in this filename"` |
| 3 → 4 | Hidden files (dotfiles) | `ls -a` then `cat .hidden` |

## 🔗 How This Connects

- Host-only networking set up today is assumed for **every** lab this month — get it wrong
  once and you risk a live scan or malware sample touching the real internet.
- The SUID/permission bits explained here are the exact mechanism abused for privilege
  escalation on **Day 9**.
- SSH key-based auth here is reused for every remote lab connection through **Day 20**.

## ✅ Checkpoint / Deliverable

- [ ] Screenshot of a working host-only VM network (VM ↔ host ping succeeds, VM ↔ internet fails)
- [ ] First successful SSH connection to a lab/Bandit host, with key-based auth configured
