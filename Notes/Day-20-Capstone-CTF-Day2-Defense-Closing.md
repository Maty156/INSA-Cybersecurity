# Day 20 — Capstone CTF: Day 2 (Defense) & Closing

**Date:** Aug 21, 2026
**Week:** Week 4 — Defense, Mobile & Capstone

---

## 🎯 Overview

The defensive half of the capstone, plus program closing. Today applies **Day 16-17**'s
SIEM/DFIR skills to triage the exact attack activity generated during **Day 19** — closing
the attacker→defender loop that's been building since Week 4 opened.

## 🔵 Blue-Team Log Triage

Working from the same Wazuh deployment (**Day 16**) monitoring the range used yesterday:

```
1. Pull alerts from the capture window covering Day 19's engagement
2. For each alert, correlate against raw logs (Day 17's Sysmon/auth.log/Event ID technique)
3. Reconstruct: what happened, in what order, on which host
4. Identify: initial access vector, privilege escalation used, any lateral movement/pivoting
```
The custom alert rule written on Day 16 (for the persistence technique built on Day 14)
should be one of the first things checked — did it actually fire against yesterday's
real attack traffic, or does it need tuning?

## 🕐 Reconstructing the Attacker's Timeline

Using the exact timeline-building method from **Day 17**:

```
[Recon phase]     Nmap scan pattern detected against multiple hosts in rapid succession
[Initial access]   SQLi payload in web logs (Day 7 technique) matching a specific parameter
[Priv-esc]           New process spawned matching a known GTFOBins pattern (Day 9)
[Lateral movement]    Kerberoast-pattern TGS requests for multiple SPNs in a short window (Day 10)
[Objective]            File access/exfil event on the final target host
```
Every row is identified using a Day 16-17 technique, describing an action from a Day
6-10/19 technique — this is the full loop closing.

## 📄 Writing the Incident Report

A real IR report format, not a casual CTF write-up:

```
1. Executive Summary   — 2-3 sentences, non-technical, for a manager-level reader
2. Timeline              — the reconstructed chronological sequence (above)
3. Indicators of Compromise (IOCs) — IPs, file hashes, registry keys, account names involved
4. Root Cause               — the actual initial access vector and why it worked
5. Impact                     — what was accessed/compromised
6. Recommendations               — specific, actionable fixes (patch X, disable Y, add rule Z)
```
This structure mirrors real SOC/IR deliverables, not just a CTF flag list — it's meant to
be handed to someone who wasn't in the room and still understand exactly what happened.

## 🎤 Final Presentation & Closing

- Team presentations walking through the full attack chain and the defense/detection story
- Program awards
- Certification

## 🔗 How This Connects

- This closes the loop opened on **Day 16** — defending against an attack chain the same
  team may have personally executed the day before, using tools/rules built specifically
  for it.
- The incident report format here is the same one referenced conceptually throughout
  **Day 15** (MITRE ATT&CK mapping) and **Day 17** (timeline building) — today is where all
  of it comes together into one deliverable.

## ✅ Checkpoint / Deliverable

- [ ] Final incident report covering Day 19's engagement, following the executive-summary → timeline → IOCs → root cause → recommendations structure
- [ ] Team presentation slides summarizing the full attack chain and defense response
