# Day 03 — Cryptography & Web Fundamentals

**Date:** Jul 29, 2026  
**Week:** Week 1: Foundations  

---

## 🎯 Overview

Core crypto primitives and how HTTP/TLS actually work under the hood, inspected live with Burp Suite.

## 📚 Topics Covered

- Hashing (SHA-256) vs encryption — one-way vs reversible
- TLS/HTTPS handshake, certificate chains, PKI (CA, public/private keypairs)
- Base64 encoding (NOT encryption) and where it shows up in web traffic/cookies
- HTTP request/response structure: methods, headers, status codes
- Intercepting and modifying requests with Burp Suite proxy

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **Burp Suite** | Intercepting proxy for inspecting/modifying HTTP(S) traffic |
| **openssl** | Generating hashes, inspecting certificates |
| **base64** | Encoding/decoding CLI utility |

## 💻 Key Commands

```bash
echo -n 'text' | sha256sum
openssl s_client -connect host:443
echo 'dGVzdA==' | base64 -d
```

## 🔗 How This Connects

- Base64 vs encryption confusion is one of the most common junior-pentester mistakes — call it out.
- Understanding the TLS handshake here explains why MITM tooling (Day 3 Burp) works on HTTP but needs a trusted cert for HTTPS.
- Hashing concepts return in Day 15 for malware/IOC hashing (MD5/SHA256 of samples).

## 📎 Resources

- Resources/CRC.Press.Crypto.Nov.2014.ISBN.1482228890.pdf
- Resources/0453-hacking-secret-ciphers-with-python.pdf

## ✅ Checkpoint / Deliverable

- [ ] Burp Suite capture of a login request + a short write-up of the TLS handshake steps observed.

---
