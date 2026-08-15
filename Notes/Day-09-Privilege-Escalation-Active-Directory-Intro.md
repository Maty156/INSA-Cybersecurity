# Day 09 — Privilege Escalation & Active Directory Intro

**Date:** Aug 06, 2026  
**Week:** Week 2: Penetration Testing & Offensive Security  

---

## 🎯 Overview

Escalating from a foothold to higher privileges on both Linux and Windows, then a first look at Active Directory as an attack surface.

## 📚 Topics Covered

- Linux priv-esc: SUID/SGID binaries (`find / -perm -4000`), writable cron jobs, `sudo -l` misconfigs
- Windows priv-esc: token impersonation (Potato-family attacks)
- AD architecture: forest → domain → OU → objects (users/computers/groups)
- Kerberos authentication flow: AS-REQ/REP, TGT, TGS
- LDAP as the AD query protocol
- NTLM authentication and why it's weaker than Kerberos

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **GTFOBins** | Reference for abusing SUID binaries for priv-esc |
| **PowerView / SharpHound** | Enumerating AD objects and relationships |
| **ldapsearch** | Querying AD over LDAP |

## 💻 Key Commands

```bash
find / -perm -4000 2>/dev/null
sudo -l
ldapsearch -x -h dc01 -b 'dc=corp,dc=com'
```

## 🔗 How This Connects

- SUID bits abused here are the exact permission bits introduced conceptually on Day 1.
- Kerberos/LDAP fundamentals here are required before Kerberoasting makes sense on Day 10.
- This day is the hinge between 'compromise one box' (Week 2 so far) and 'compromise a domain' (Day 10).

## 📎 Resources

- https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/

## ✅ Checkpoint / Deliverable

- [ ] Documented Linux priv-esc path (user → root) on the lab VM.

---
