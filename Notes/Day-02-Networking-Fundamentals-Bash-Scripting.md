# Day 02 — Networking Fundamentals & Bash Scripting

**Date:** Jul 28, 2026  
**Week:** Week 1: Foundations  

---

## 🎯 Overview

OSI/TCP-IP theory paired with the two tools (Wireshark, Nmap) used in nearly every later offensive day, plus Bash automation.

## 📚 Topics Covered

- OSI 7-layer model vs TCP/IP 4-layer model, encapsulation
- IP addressing & subnetting (CIDR, VLSM)
- DNS resolution flow (recursive vs authoritative)
- Common ports/services (21,22,23,25,53,80,443,445,3389...)
- Packet capture and filtering in Wireshark
- Nmap scan types: `-sS`, `-sT`, `-sU`, `-sV`, `-A`
- Bash scripting: variables, loops, conditionals, `$()`, exit codes — for automating recon

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **Wireshark** | Traffic inspection, filtering with display filters (`tcp.port==80`) |
| **Nmap** | Host/port discovery and service fingerprinting |
| **Bash** | Automating repetitive recon/enumeration steps |

## 💻 Key Commands

```bash
nmap -sV -A target
tcpdump -i eth0 -w cap.pcap
for ip in $(cat hosts.txt); do ping -c1 $ip; done
```

## 🔗 How This Connects

- Subnetting math shows up again wherever you need to scope a pentest engagement.
- A Bash for-loop over Nmap is the seed of every recon script you'll write this month.
- Wireshark filters here directly feed into Day 17 (Wazuh/Sysinternals log correlation).

## 📎 Resources

- Resources/networking.pdf
- https://www.professormesser.com/network-plus/n10-009/

## ✅ Checkpoint / Deliverable

- [ ] Nmap scan of the lab range + a short Bash script that automates it.

---
