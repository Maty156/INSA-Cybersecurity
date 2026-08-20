# Day 19 — Capstone CTF: Day 1 (Offensive Phase)

**Date:** Aug 20, 2026
**Week:** Week 4 — Defense, Mobile & Capstone

---

## 🎯 Overview

The offensive half of the 2-day team capstone: applying four weeks of material end-to-end
against a live range, under time pressure, as a team. No new tools today — this is pure
execution of everything from Day 1 through Day 18.

## 🗺️ Applying the Day 6 Methodology First

Before touching anything, the same discipline from **Day 6** applies at full scale:
```
1. Scope the range (which hosts/subnets are actually in play?)
2. Passive + active recon across every discovered host (Day 2 Nmap, Day 6 SMB/SNMP enum)
3. Build a target profile per host BEFORE attempting exploitation
4. Prioritize targets by likely impact (a DC > a random workstation)
```
Teams that skip straight to exploitation on the first host they see typically waste more
time than they save — the Day 6 habit pays off most exactly when the clock is running.

## 🔗 Chaining the Full Skill Stack

A realistic path through a range like this touches almost every prior week:

```
Recon (Day 6)
  → Web app found on one host: injection/auth flaws (Day 7-8) → initial foothold
  → Foothold used for AD enumeration (Day 9): LDAP/SMB reveals domain structure
  → Kerberoasting or a misconfigured ACL (Day 10) → higher-privilege domain account
  → BloodHound path to a second host holding a suspicious binary
  → Static + dynamic RE (Day 11-12) on that binary reveals a hardcoded credential/flag
  → Pivot (Day 10, Ligolo-ng) into a segment holding the final objective host
```
Every "how do I do X" question today should map back to a specific earlier day's notes —
that's the whole point of having them written down.

## 🚩 Objective Categories

Based on the program's stated capstone scope, expect a mix of:
- **Web exploitation** objectives (Week 2, Days 7-8 skillset)
- **Active Directory compromise** objectives (Days 9-10)
- **Reverse engineering** objectives (Days 11-12, possibly Day 13-15 if a "malware"-themed
  flag is included)

## 📝 Evidence Collection Discipline

Every flag captured needs supporting evidence for tomorrow's report — capture as you go,
not after:
- Full command used (not just "I ran sqlmap")
- Screenshot of the actual flag/output
- Timestamp (feeds directly into Day 20's incident-report timeline, viewed from the other
  side)

## 🔗 How This Connects

- This is a direct, cumulative application of **Days 1-18** — no new tools, just execution
  under time pressure, exactly as the program's daily-deliverable structure has been
  building toward.
- Every artifact captured today (screenshots, hashes, payloads, timestamps) is the raw
  material for **Day 20**'s incident report and closing presentation.

## ✅ Checkpoint / Deliverable

- [ ] Captured flags for each objective attempted, with the technique/day it maps to noted
- [ ] Supporting evidence (screenshots, commands, timestamps) for every captured flag
