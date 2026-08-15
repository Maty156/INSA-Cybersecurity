# Day 10 — Active Directory Attacks & Pivoting

**Date:** Aug 07, 2026  
**Week:** Week 2: Penetration Testing & Offensive Security  

---

## 🎯 Overview

Week 2's capstone day: mapping and attacking a live AD environment, then pivoting deeper into the network from a single compromised host.

## 📚 Topics Covered

- BloodHound: collecting with SharpHound, graphing attack paths to Domain Admin
- Kerberoasting — requesting service tickets and cracking them offline for service-account passwords
- AS-REP Roasting — targeting accounts with Kerberos pre-auth disabled
- Mimikatz: dumping credentials/hashes from LSASS memory
- Lateral movement techniques (pass-the-hash, pass-the-ticket)
- Pivoting through a compromised host into an otherwise unreachable network segment with Ligolo-ng

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **BloodHound** | Visual attack-path mapping across an AD domain |
| **Impacket (GetUserSPNs.py, secretsdump.py)** | Kerberoasting and credential extraction |
| **Ligolo-ng** | Tunnelling/pivoting into internal segments |

## 💻 Key Commands

```bash
GetUserSPNs.py corp.com/user -request
mimikatz # sekurlsa::logonpasswords
ligolo-ng: proxy + agent tunnel setup
```

## 🔗 How This Connects

- This day operationalises everything from Day 9 (Kerberos/LDAP theory) into an actual compromise chain.
- Ligolo-ng pivoting here is the same 'expand your foothold' logic behind Day 17's lateral-movement forensic timelines.
- BloodHound attack paths are exactly what the Day 20 defense team should be hunting for in reverse.

## 📎 Resources

- https://github.com/SpecterOps/BloodHound
- https://github.com/fortra/impacket
- https://www.netexec.wiki/

## ✅ Checkpoint / Deliverable

- [ ] BloodHound attack path screenshot + one cracked Kerberoast hash from the lab domain.

---
