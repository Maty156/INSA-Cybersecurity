# Day 02 — Networking Fundamentals & Bash Scripting

**Date:** Jul 28, 2026
**Week:** Week 1 — Foundations

---

## 🎯 Overview

Everything offensive or defensive in this program rides on top of a network, so today
built the full theory stack from OSI down to real packets in Wireshark, then Nmap for
active discovery, then Bash to start automating recon instead of typing commands by hand.

## 🧱 The OSI Model (7 Layers)

The OSI model describes how data moves from an application on one machine to an
application on another, broken into 7 conceptual layers. Data gets wrapped ("encapsulated")
in a header at each layer going down, and unwrapped going back up on the receiving end.

| # | Layer | Unit (PDU) | What happens here | Example protocols |
|---|-------|-----------|--------------------|--------------------|
| 7 | Application | Data | What the user/program actually interacts with | HTTP, DNS, SSH, FTP |
| 6 | Presentation | Data | Formatting, encryption, compression | TLS, JPEG, ASCII |
| 5 | Session | Data | Establishing/managing/tearing down sessions | NetBIOS, RPC |
| 4 | Transport | Segment (TCP) / Datagram (UDP) | End-to-end delivery, ports, reliability | TCP, UDP |
| 3 | Network | Packet | Logical addressing, routing between networks | IP, ICMP |
| 2 | Data Link | Frame | Physical addressing on the local segment | Ethernet, ARP, MAC |
| 1 | Physical | Bits | Actual electrical/radio/optical signal | Cabling, Wi-Fi radio |

**Encapsulation, top to bottom:** an HTTP request (L7) gets a TCP header added (L4,
becomes a segment) → an IP header added (L3, becomes a packet) → an Ethernet header +
trailer added (L2, becomes a frame) → converted to electrical signal (L1) and sent out
the wire.

The simpler **TCP/IP 4-layer model** used in practice maps like this:

| TCP/IP Layer | Maps to OSI Layers |
|---|---|
| Application | 7, 6, 5 |
| Transport | 4 |
| Internet | 3 |
| Network Access | 2, 1 |

**Why this matters for security:** knowing the layer a tool/attack operates at tells you
what it can and can't see. Wireshark works at L2-L7 (it's watching raw frames). Nmap's TCP
scans work at L3-L4. An ARP spoofing attack is purely L2. A phishing email is purely L7.

## 🌐 IP Addressing & Subnetting

An IPv4 address is 32 bits, written as 4 decimal octets (`192.168.1.10`). A **subnet mask**
(or CIDR suffix, e.g. `/24`) splits that 32 bits into a **network portion** and a **host
portion**.

`192.168.1.0/24` means: the first 24 bits are the network, the last 8 bits are host
addresses.

- Network address: `192.168.1.0` (all host bits 0)
- Broadcast address: `192.168.1.255` (all host bits 1)
- Usable host range: `192.168.1.1` – `192.168.1.254` → **254 usable hosts**

### Common CIDR / mask cheat sheet

| CIDR | Subnet Mask | Usable Hosts |
|------|-------------|--------------|
| /24 | 255.255.255.0 | 254 |
| /25 | 255.255.255.128 | 126 |
| /26 | 255.255.255.192 | 62 |
| /27 | 255.255.255.224 | 30 |
| /30 | 255.255.255.252 | 2 (point-to-point links) |

### Worked VLSM example

Given `192.168.10.0/24`, split it into 4 equal subnets for 4 departments:

```
192.168.10.0/26   → 192.168.10.0   - 192.168.10.63    (62 usable)
192.168.10.64/26  → 192.168.10.64  - 192.168.10.127   (62 usable)
192.168.10.128/26 → 192.168.10.128 - 192.168.10.191   (62 usable)
192.168.10.192/26 → 192.168.10.192 - 192.168.10.255   (62 usable)
```
Each `/26` borrows 2 extra bits from the host portion (`/24` → `/26`), quartering the
address space (2² = 4 subnets) while quartering host capacity per subnet.

## 🧭 DNS Resolution Flow

When you request `example.com`:

1. **Stub resolver** (your OS) checks its local cache — if cached, done.
2. Otherwise it asks a **recursive resolver** (e.g. your ISP's DNS, or `8.8.8.8`).
3. The recursive resolver queries a **root server** → gets referred to the `.com` **TLD
   server**.
4. The TLD server refers it to `example.com`'s **authoritative name server**.
5. The authoritative server returns the actual A/AAAA record (the IP).
6. The recursive resolver caches and returns the answer to you.

```bash
dig example.com               # full resolution, shows answer + query time
dig +trace example.com        # shows every hop above manually
nslookup example.com
```

## 🔌 Common Ports & Services

| Port | Protocol | Service |
|------|----------|---------|
| 21 | TCP | FTP (control) |
| 22 | TCP | SSH |
| 23 | TCP | Telnet (unencrypted — red flag if seen) |
| 25 | TCP | SMTP (mail) |
| 53 | TCP/UDP | DNS |
| 80 | TCP | HTTP |
| 443 | TCP | HTTPS |
| 445 | TCP | SMB (Windows file sharing) |
| 3389 | TCP | RDP |

Knowing this table by heart is what makes an Nmap scan output immediately readable instead
of a wall of numbers.

## 🦈 Wireshark — Packet Capture & Filtering

Wireshark captures raw frames off an interface and lets you inspect every OSI layer in
one packet.

```bash
# capture on the command line with tcpdump, open later in Wireshark
sudo tcpdump -i eth0 -w capture.pcap
```

**Display filters** (typed into the Wireshark filter bar, NOT capture filters):

```
tcp.port == 80                 # only HTTP traffic
ip.addr == 192.168.1.10        # only traffic to/from this host
http.request.method == "POST"  # only POST requests
dns                             # only DNS traffic
tcp.flags.syn == 1 && tcp.flags.ack == 0   # only SYN packets (start of TCP handshake)
```

**TCP 3-way handshake**, visible in any capture:
```
Client → Server:  SYN
Server → Client:  SYN, ACK
Client → Server:  ACK
```
Recognizing this pattern is the baseline for spotting a normal connection vs. a scan
(Nmap's SYN scan deliberately never completes the handshake — see below).

## 🎯 Nmap — Active Scanning

| Scan type | Flag | What it does |
|-----------|------|----------------|
| TCP Connect | `-sT` | Completes full 3-way handshake — reliable, but noisy/logged |
| SYN ("half-open") | `-sS` | Sends SYN, reads SYN-ACK, never sends final ACK — faster, stealthier, needs root |
| UDP | `-sU` | Probes UDP ports (slower — no handshake to rely on) |
| Version detection | `-sV` | Grabs service banners to identify software/version |
| Aggressive | `-A` | Combines `-sV`, OS detection, traceroute, and default scripts |

```bash
nmap -sS -sV -p- 192.168.1.10        # SYN scan, version detect, ALL 65535 ports
nmap -A 192.168.1.0/24                # aggressive scan across a whole subnet
nmap -sU -p 53,161 192.168.1.10       # UDP scan on specific ports
```

**Example (annotated) output:**
```
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.9p1 Ubuntu
80/tcp   open  http    Apache httpd 2.4.52
445/tcp  open  microsoft-ds Samba smbd 4.15
```
Every line here is a lead: OpenSSH 8.9p1 → check for known CVEs; Samba → straight into
Day 6's SMB enumeration.

## 🐚 Bash Scripting for Automation

```bash
#!/bin/bash
# Simple host-sweep: ping every address in a subnet, list the live ones

subnet="192.168.1"
for i in $(seq 1 254); do
  ip="${subnet}.${i}"
  if ping -c 1 -W 1 "$ip" &>/dev/null; then
    echo "$ip is up"
  fi
done
```

Key building blocks used above:
- `for i in $(seq 1 254)` — loop, `$()` captures command output into a variable
- `ping -c 1 -W 1` — 1 packet, 1-second timeout, so the loop doesn't stall
- `&>/dev/null` — discard both stdout and stderr, we only care about the exit code
- `if [ ... ]; then` / `$?` — exit-code-based conditionals are how Bash scripts make
  decisions

```bash
# Feed a live-host list straight into Nmap
for ip in $(cat live_hosts.txt); do
  nmap -sV -oN "scan_${ip}.txt" "$ip"
done
```

## 🔗 How This Connects

- The subnetting math here is exactly what's used to scope an engagement's target range on
  **Day 6**.
- This Bash for-loop-over-Nmap pattern is the seed of every recon/enumeration script
  written for the rest of Week 2.
- Wireshark filters built today are reused directly for log/traffic correlation in
  **Day 17** (DFIR & log analysis).
- The 3-way handshake understood here is *why* Nmap's SYN scan (`-sS`) is stealthier than
  a Connect scan (`-sT`) — it never completes the handshake a firewall/IDS might log.

## 📎 Resources

- `Resources/networking.pdf`
- https://www.professormesser.com/network-plus/n10-009/n10-009-video/n10-009-training-course/

## ✅ Checkpoint / Deliverable

- [ ] Full Nmap scan (`-sV -A`) of the lab range, saved to a file
- [ ] Bash script that pings a subnet and outputs only the live hosts
- [ ] One Wireshark capture with at least 3 different display filters applied and screenshotted
