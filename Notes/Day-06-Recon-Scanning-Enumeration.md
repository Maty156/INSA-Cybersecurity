# Day 06 — Reconnaissance, Scanning & Enumeration

**Date:** Aug 03, 2026
**Week:** Week 2 — Penetration Testing & Offensive Security

---

## 🎯 Overview

Opening Week 2 with formal methodology first, then the deep scanning/enumeration work
that every later attack chain in this program starts from. The habit built today —
recon before touching anything — is the backbone of every offensive day from here to
the capstone.

## 🗂️ Pentest Methodology

A standard engagement moves through five phases:

```
1. Reconnaissance     → passive/active info gathering, no direct interaction where possible
2. Scanning            → active probing: port scans, service enumeration
3. Exploitation         → turning a discovered weakness into access
4. Post-Exploitation      → priv-esc, lateral movement, data of interest
5. Reporting              → what was found, impact, remediation
```

**Passive vs active recon:**
- **Passive** — no direct traffic to the target: WHOIS lookups, DNS records, search
  engine results, LinkedIn/job postings for tech-stack hints, `crt.sh` for certificate
  transparency logs (subdomain discovery without touching the target).
- **Active** — direct interaction: port scans, banner grabs, spidering the web app. Louder
  and more likely to be logged/detected.

## 📋 Scoping & Rules of Engagement (RoE)

Before any active scanning:
- **In-scope** IP ranges/domains, explicitly listed
- **Out-of-scope** systems (e.g. third-party-hosted infra you don't have permission for)
- **Testing window** (specific dates/times, especially for production systems)
- **Emergency contact** in case something breaks
- **Allowed techniques** — is social engineering in scope? DoS testing?

This is the same authorization principle from **Day 1**'s ethics discussion, made
concrete as a real document.

## 🎯 Nmap Deep Scanning

Beyond Day 2's basic scans, today goes into version detection depth and the NSE
scripting engine.

```bash
nmap -sV --version-intensity 9 target      # aggressive version fingerprinting
nmap --script vuln target                  # run all "vuln" category NSE scripts
nmap --script smb-enum-shares,smb-enum-users -p445 target
nmap -T4 -p- target                        # all 65535 ports, faster timing template
```

**Timing templates** (`-T0` paranoid → `-T5` insane) trade speed for stealth — `-T4` is
the common default for lab/CTF work where IDS evasion isn't the goal.

**NSE (Nmap Scripting Engine)** categories worth knowing: `auth`, `default`, `discovery`,
`intrusive`, `safe`, `vuln`. Scripts live in `/usr/share/nmap/scripts/` and can be listed
with `nmap --script-help <name>`.

## 📁 SMB Enumeration

SMB (port 445) is one of the highest-value services to enumerate — misconfigured shares
are a constant real-world finding.

```bash
smbclient -L //target -N            # list shares, -N = no password (anonymous)
smbclient //target/share -N          # connect to a specific share anonymously
enum4linux -a target                  # all-in-one: shares, users, groups, policy
smbmap -H target                      # shares + read/write permission matrix
```

**Example finding:** an anonymous, world-writable share is a direct path to dropping a
payload — and a common real entry point later chained into the **Day 9-10** AD attacks.

## 📡 SNMP Enumeration

SNMP (port 161/UDP) often runs with default/guessable "community strings" (its
password equivalent) — `public` (read) and `private` (read-write) are default on many
devices.

```bash
snmpwalk -v2c -c public target                      # walk the entire MIB tree
snmpwalk -v2c -c public target 1.3.6.1.2.1.1.5.0     # query a specific OID (sysName)
onesixtyone -c community_strings.txt target           # brute-force community strings
```
A readable SNMP tree can leak running processes, installed software, routing tables, and
sometimes even usernames — pure gold for building a target profile.

## 🧩 Building a Target Profile

By the end of recon/enumeration, a solid profile looks like:

```
Host: 192.168.1.10
OS: Ubuntu 22.04 (via TTL + Nmap OS detection)
Open ports: 22 (OpenSSH 8.9p1), 80 (Apache 2.4.52), 139/445 (Samba 4.15)
SMB shares: \\192.168.1.10\public (anonymous, writable)
Notable: Samba version has a known CVE — candidate for exploitation phase
```
This profile is what phase 3 (exploitation) is built from — nothing gets attacked blind.

## 🔗 How This Connects

- This methodology is the checklist every offensive day for the rest of the program
  silently follows — recon first, always.
- Anonymous SMB shares found here are frequently the actual entry point exploited in
  **Day 9-10**'s AD attacks.
- A disciplined scope/RoE habit built here is exactly what the **Day 17** incident report
  references from the defender's side.
- The target-profile format here becomes the "Recon" section of every CTF write-up in
  `Challenges/`.

## 📎 Resources

- `Resources/Day1_Penetration_Testing_Methodology.pdf`

## ✅ Checkpoint / Deliverable

- [ ] Full target profile (open ports, service versions, OS guess, SMB/SNMP findings) for the assigned lab host
- [ ] Nmap NSE `vuln` scan output with at least one flagged finding
