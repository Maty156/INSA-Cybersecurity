# Day 17 — DFIR & Log Analysis

**Date:** Aug 18, 2026  
**Week:** Week 4: Defense, Mobile & Capstone  

---

## 🎯 Overview

Digital Forensics & Incident Response — reconstructing what happened on a host after the fact from logs alone.

## 📚 Topics Covered

- Incident response lifecycle: preparation → identification → containment → eradication → recovery → lessons learned
- Sysinternals suite for live triage (Autoruns, Process Explorer, Sigcheck)
- Windows Event Logs (`.evtx`) — key Event IDs for logon (4624/4625), process creation (4688), etc.
- Linux log analysis (`/var/log/auth.log`, `syslog`, `journalctl`)
- Building a timeline that reconstructs the attacker's actions in chronological order

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **Sysinternals** | Live Windows host triage utilities |
| **evtx dump tools** | Parsing Windows Event Logs |
| **journalctl / grep / awk** | Parsing Linux logs at the command line |

## 💻 Key Commands

```bash
journalctl -u sshd --since '1 hour ago'
grep 'Failed password' /var/log/auth.log
evtx_dump.py Security.evtx | grep 4624
```

## 🔗 How This Connects

- This is the log-based half of Day 15's memory-forensics work, applied to disk/event logs instead.
- The Event IDs learned here are exactly what Day 16's Wazuh rules should be alerting on.
- Timeline-building here is the direct input to the Day 20 incident report and closing presentation.

## ✅ Checkpoint / Deliverable

- [ ] A written incident timeline reconstructing one simulated intrusion from logs alone.

---
