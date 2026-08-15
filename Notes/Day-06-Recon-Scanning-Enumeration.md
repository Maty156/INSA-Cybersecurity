# Day 06 — Reconnaissance, Scanning & Enumeration

**Date:** Aug 03, 2026  
**Week:** Week 2: Penetration Testing & Offensive Security  

---

## 🎯 Overview

Opening Week 2 with formal pentest methodology and the deep scanning/enumeration that every later attack chain starts from.

## 📚 Topics Covered

- Pentest methodology & phases (recon → scanning → exploitation → post-exploitation → reporting)
- Scoping and rules of engagement
- Nmap deep scanning: version detection, NSE scripts, timing templates
- SMB enumeration (`smbclient`, `enum4linux`, `smbmap`)
- SNMP enumeration (`snmpwalk`, community strings)
- Building a target profile from scan output before touching exploitation

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **Nmap NSE** | Scripted enumeration (`--script smb-enum-shares`) |
| **enum4linux** | SMB/NetBIOS enumeration |
| **snmpwalk** | Walking an SNMP MIB tree |

## 💻 Key Commands

```bash
nmap -p445 --script smb-enum-shares,smb-enum-users target
enum4linux -a target
snmpwalk -v2c -c public target
```

## 🔗 How This Connects

- This day's methodology is the checklist every Week 2-4 offensive day silently follows.
- Anonymous SMB shares found here are frequently the actual entry point exploited in Day 9-10 AD attacks.
- A disciplined scope/RoE habit here is what your Day 17 incident report on the other side references.

## 📎 Resources

- Resources/Day1_Penetration_Testing_Methodology.pdf

## ✅ Checkpoint / Deliverable

- [ ] Full target profile (open ports, services, versions, SMB/SNMP findings) for the assigned lab host.

---
