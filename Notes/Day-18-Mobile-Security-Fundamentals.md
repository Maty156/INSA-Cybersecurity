# Day 18 — Mobile Security Fundamentals

**Date:** Aug 19, 2026  
**Week:** Week 4: Defense, Mobile & Capstone  

---

## 🎯 Overview

A focused day on Android security — static decompilation, dynamic instrumentation, and bypassing common client-side protections.

## 📚 Topics Covered

- Android architecture: APK structure, Dalvik/ART, permission model
- Static analysis with JADX — decompiling an APK back to readable Java/Kotlin-like source
- Repackaging/inspecting an APK with Apktool
- Dynamic instrumentation with Frida/objection — hooking functions at runtime
- Bypassing client-side checks: root detection, SSL pinning

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **JADX** | APK → readable source decompiler |
| **Apktool** | APK decode/rebuild for static inspection and repackaging |
| **Frida + objection** | Runtime hooking/instrumentation, SSL-pinning bypass |

## 💻 Key Commands

```bash
jadx-gui app.apk
apktool d app.apk
objection -g com.target.app explore  # then: android sslpinning disable
```

## 🔗 How This Connects

- JADX decompilation here is the same static-analysis instinct from Day 11, applied to Java bytecode instead of native binaries.
- SSL-pinning bypass here reconnects to Day 3's TLS/PKI theory — you're defeating the exact trust chain studied there.
- This day's client-bypass techniques are a plausible attack surface for the Day 19-20 capstone if a mobile component is in scope.

## 📎 Resources

- Resources/mobile security.pptx
- https://github.com/skylot/jadx
- https://github.com/iBotPeaches/Apktool
- https://github.com/frida/frida
- https://github.com/cd80-ctf/InjuredAndroidWriteups

## ✅ Checkpoint / Deliverable

- [ ] SSL-pinning bypass (or equivalent client-side check bypass) demonstrated on a test APK.

---
