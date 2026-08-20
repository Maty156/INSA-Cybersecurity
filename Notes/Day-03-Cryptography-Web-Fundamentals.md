# Day 03 — Cryptography & Web Fundamentals

**Date:** Jul 29, 2026
**Week:** Week 1 — Foundations

---

## 🎯 Overview

Two halves: crypto primitives (hashing, TLS/PKI) that show up constantly in later
weeks, and HTTP/web mechanics inspected live in Burp Suite — the tool used for the rest
of the web-pentesting track.

## #️⃣ Hashing vs Encryption

These get confused constantly — they solve different problems.

| | Hashing | Encryption |
|---|---------|------------|
| Direction | One-way (can't reverse) | Two-way (can decrypt with the key) |
| Purpose | Integrity/verification (passwords, file checksums) | Confidentiality (protecting data in transit/at rest) |
| Example | SHA-256, MD5, bcrypt | AES, RSA |
| Same input → same output? | Always | Only with the same key/IV |

```bash
echo -n "password123" | sha256sum
# ef92b778bafe771e89245b89ecbc08a44a4e166c06659911881f383d4473e94

echo -n "password123" | md5sum
# 482c811da5d5b4bc6d497ffa98491e38
```
**Avalanche effect** — changing one character completely changes the hash:
```bash
echo -n "password124" | sha256sum
# totally different output, not "close" to the one above
```
This is why hashes are used to verify file integrity (a 1-byte malware edit produces a
completely different hash — this comes back on **Day 15** for IOC hashing) and why
passwords are stored as hashes, never plaintext.

## 🔒 TLS / HTTPS & PKI

**TLS handshake (simplified, TLS 1.2-style):**
```
1. Client Hello    — client proposes supported cipher suites
2. Server Hello    — server picks a cipher suite, sends its certificate
3. Certificate check — client verifies the cert against a trusted CA chain
4. Key exchange    — client & server derive a shared symmetric session key
5. Finished        — both sides switch to encrypted communication
```

**PKI (Public Key Infrastructure):**
- Every HTTPS server has a **keypair**: a public key (in its certificate) and a private
  key (kept secret on the server).
- A **Certificate Authority (CA)** — e.g. Let's Encrypt, DigiCert — signs the server's
  certificate, vouching "this public key really belongs to example.com".
- Your browser trusts a built-in list of **root CAs**. It verifies the chain:
  `server cert → intermediate CA → root CA` (all found in its trust store).

```bash
openssl s_client -connect example.com:443 -showcerts   # dump the full cert chain
openssl x509 -in cert.pem -text -noout                  # inspect one certificate
```

**Why this matters offensively:** intercepting HTTP traffic is trivial (Wireshark
already showed you that on Day 2). Intercepting HTTPS requires either (a) the client
trusting a certificate you control (Burp's proxy CA, installed in the test browser) or
(b) breaking the crypto itself — practically infeasible. This is the exact trust chain
defeated deliberately on **Day 18** (SSL-pinning bypass on mobile).

## 🔤 Base64 — NOT Encryption

Base64 encodes binary data into printable ASCII. It is **fully reversible with no key** —
it provides zero confidentiality, only a transport-safe format.

```bash
echo -n "admin:password" | base64
# YWRtaW46cGFzc3dvcmQ=
echo "YWRtaW46cGFzc3dvcmQ=" | base64 -d
# admin:password
```
You'll see Base64 everywhere: HTTP Basic Auth headers, JWT segments (**Day 8**), cookies.
Seeing Base64 in traffic is never itself "encrypted" — always try decoding it first.

## 🌐 HTTP Request/Response Structure

**Request:**
```
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Cookie: session=abc123

username=admin&password=secret
```
**Response:**
```
HTTP/1.1 200 OK
Content-Type: text/html
Set-Cookie: session=xyz789

<html>...</html>
```

| Method | Purpose |
|--------|---------|
| GET | Retrieve a resource (should have no side effects) |
| POST | Submit data (create/process) |
| PUT | Replace a resource |
| DELETE | Remove a resource |

| Status code range | Meaning |
|---|---|
| 2xx | Success (200 OK, 201 Created) |
| 3xx | Redirect (301, 302) |
| 4xx | Client error (401 Unauthorized, 403 Forbidden, 404 Not Found) |
| 5xx | Server error (500 Internal Server Error) |

## 🕵️ Burp Suite — Intercepting Proxy

Burp sits between your browser and the target as a proxy, letting you see and modify
every request before it's sent.

1. Set browser proxy to `127.0.0.1:8080` (Burp's default listener).
2. Install Burp's CA certificate in the browser so HTTPS interception doesn't throw
   trust errors.
3. **Proxy → Intercept** — pause a request, edit it, forward it.
4. **Repeater** — resend a modified request repeatedly, tweaking parameters each time.
5. **Intruder** — automate fuzzing a parameter against a wordlist.

```
Example: intercepted login POST, changed username=admin to username=admin' -- 
to start probing for SQL injection (full technique on Day 7)
```

## 🔗 How This Connects

- Base64-vs-encryption confusion resolved here prevents a classic junior mistake spotting
  "encrypted" data in every later web-pentesting day.
- The TLS/PKI trust chain explained here is the exact mechanism bypassed in **Day 18**
  (SSL pinning).
- Hashing concepts return for malware sample identification (MD5/SHA256) on **Day 15**.
- Burp Suite set up today is the primary tool for all of **Day 7–8** (Web Pentesting I & II).

## 📎 Resources

- `Resources/CRC.Press.Crypto.Nov.2014.ISBN.1482228890.pdf`
- `Resources/0453-hacking-secret-ciphers-with-python.pdf`

## ✅ Checkpoint / Deliverable

- [ ] Burp Suite capture of a login request, with the CA cert successfully installed
- [ ] Short write-up of the 5-step TLS handshake as observed via `openssl s_client`
- [ ] One Base64-encoded value decoded manually from captured traffic
