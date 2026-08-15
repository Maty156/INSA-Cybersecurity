# Day 01 — Kickoff, Ethics & Linux Fundamentals

**Date:** Jul 27, 2026  
**Week:** Week 1: Foundations  

---

## 🎯 Overview

Program kickoff — ethics sign-off, VM setup, and the Linux fundamentals every later module builds on.

## 📚 Topics Covered

- Code of conduct / ethics agreement before any hands-on work
- Hypervisor setup (VirtualBox/VMware) with host-only networking to isolate lab VMs from the host and internet
- Linux CLI basics: filesystem hierarchy, navigation, `man` pages
- File permissions: `rwx`, `chmod`, `chown`, SUID/SGID bits (foundation for later priv-esc work)
- SSH: key-based auth, `ssh-keygen`, connecting to remote lab boxes

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **VirtualBox/VMware** | Host-only adapter to isolate the attack range from the host network |
| **chmod/chown** | Managing file permissions and ownership |
| **ssh / ssh-keygen** | Remote access and key-based authentication |

## 💻 Key Commands

```bash
chmod 750 file
ssh -i id_rsa user@target
ls -la (spot SUID bits: -rwsr-xr-x)
```

## 🔗 How This Connects

- Everything after Day 1 assumes host-only, snapshot-able VMs — get this right first.
- SUID/SGID permissions here are the same bits abused in Day 9 privilege escalation.
- Ethics sign-off exists because most tools taught this month are dual-use.

## ✅ Checkpoint / Deliverable

- [ ] Isolated VM confirmed working + first SSH connection screenshot/log.

---
