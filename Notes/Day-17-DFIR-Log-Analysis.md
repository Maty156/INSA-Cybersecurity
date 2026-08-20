# Day 17 — DFIR & Log Analysis

**Date:** Aug 18, 2026
**Week:** Week 4 — Defense, Mobile & Capstone

---

## 🎯 Overview

Digital Forensics & Incident Response — reconstructing exactly what happened on a host,
after the fact, from logs and artifacts alone. This is the disk/log-based counterpart to
**Day 15**'s memory forensics.

## 🔄 Incident Response Lifecycle

```
1. Preparation      — tooling, playbooks, and access in place BEFORE an incident
2. Identification    — confirm something actually happened, scope it
3. Containment         — stop the bleeding (isolate the host, disable an account)
4. Eradication          — remove the malware/backdoor/persistence
5. Recovery               — restore normal operation, verify it's actually clean
6. Lessons Learned          — what let this happen, what changes going forward
```
This lifecycle is the formal structure the **Day 20** incident report is written against.

## 🧰 Sysinternals Suite — Live Triage

| Tool | Purpose |
|------|---------|
| **Autoruns** | Lists every autostart location at once — Run keys, Scheduled Tasks, Services (directly surfaces **Day 14**'s persistence mechanisms) |
| **Process Explorer** | Task Manager++, shows parent/child trees, loaded DLLs, verified signatures |
| **Sigcheck** | Checks a file's digital signature and (optionally) VirusTotal hash reputation |

```
Autoruns.exe   -> sort by "Not verified" first -> unsigned entries are the first suspects
sigcheck.exe -h -vt suspicious.exe   -> hash lookup against VirusTotal
```

## 📜 Windows Event Logs

Stored as `.evtx` files, queried by **Event ID** — a small set of IDs cover most
investigations:

| Event ID | Meaning |
|----------|---------|
| 4624 | Successful logon |
| 4625 | Failed logon |
| 4688 | New process created |
| 4697 | Service installed |
| 7045 | New service installed (System log) |
| 4720 | User account created |

```bash
# Parsing an exported .evtx on Linux with python-evtx
evtx_dump.py Security.evtx | grep -A 5 "EventID>4624"

# On Windows itself, PowerShell:
Get-WinEvent -LogName Security -FilterXPath "*[System[EventID=4625]]"
```
A burst of 4625 (failed logon) events followed immediately by a 4624 (success) for the
same account is the textbook signature of a successful brute-force — the exact same
pattern flagged by the SIEM correlation logic from **Day 16**, now being read manually.

## 🐧 Linux Log Analysis

```bash
grep "Failed password" /var/log/auth.log | awk '{print $11}' | sort | uniq -c | sort -rn
# counts failed SSH login attempts by source IP

journalctl -u sshd --since "2026-08-18 09:00" --until "2026-08-18 10:00"
# systemd-based systems: query the sshd service's journal for a specific window

last -f /var/log/wtmp    # login history
```

## 🕐 Building a Timeline

The core DFIR deliverable is a chronological timeline correlating events across every
available source:

```
09:14:02  SSH auth.log: 40 failed logins from 203.0.113.5 (brute force attempt)
09:14:55  SSH auth.log: successful login as 'svc-backup' from 203.0.113.5
09:15:10  Event ID 4688: cmd.exe spawned by svc-backup's session
09:15:40  Wazuh alert (Day 16 rule): Registry Run key modified
09:16:02  Sysmon: outbound connection to 198.51.100.20:443 (C2, matches Day 13's INetSim IOC)
```
Every row here draws on a technique from an earlier day — brute-force detection (Day 16
correlation logic), process-creation event IDs (this section), persistence (Day 14), and
network IOC matching (Day 13/15). The timeline IS the investigation.

## 🔗 How This Connects

- This is the log-based half of **Day 15**'s memory-forensics work, applied to disk/event
  logs instead of a RAM image.
- The Event IDs learned here are exactly what **Day 16**'s Wazuh rules should already be
  alerting on — this day is "read it manually" as a fallback/verification skill.
- Timeline-building here is the direct input to the **Day 20** incident report and closing
  presentation.

## ✅ Checkpoint / Deliverable

- [ ] A written incident timeline reconstructing one simulated intrusion, correlating at least 3 different log sources
- [ ] Autoruns scan of a compromised lab host with the malicious persistence entry identified
