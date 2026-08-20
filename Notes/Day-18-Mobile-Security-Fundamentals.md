# Day 18 — Mobile Security Fundamentals

**Date:** Aug 19, 2026
**Week:** Week 4 — Defense, Mobile & Capstone

---

## 🎯 Overview

A focused day on Android security: static decompilation, dynamic runtime instrumentation,
and bypassing the client-side checks apps rely on to protect themselves.

## 🤖 Android Architecture

```
Application layer     — the APK: your app's Java/Kotlin code, resources, manifest
Framework (Java API)    — Android SDK services (Activity Manager, etc.)
ART (Android Runtime)     — executes compiled app bytecode (replaced Dalvik since Android 5)
Native libraries + HAL      — C/C++ libs, hardware abstraction
Linux Kernel                  — the actual OS core (permissions, drivers, processes)
```

**APK structure** (an APK is just a ZIP file):
```
app.apk
├── AndroidManifest.xml   — permissions, components, entry points
├── classes.dex            — compiled bytecode (Dalvik Executable format)
├── res/                     — resources (layouts, images, strings)
└── lib/                       — native (.so) libraries, if any
```

**Permission model:** each app declares required permissions in the manifest; from
Android 6+, dangerous permissions (camera, location, contacts) are requested at *runtime*,
not just install time — the user can grant/revoke individually.

## 🔬 Static Analysis with JADX

JADX decompiles an APK's `classes.dex` bytecode back into readable Java-like source.

```bash
jadx-gui app.apk           # GUI: browse the decompiled source tree directly
jadx app.apk -d output/     # CLI: dump decompiled source to a folder
```
Workflow mirrors **Day 11**'s "strings → xref" approach: search the decompiled source for
interesting strings (API endpoints, hardcoded keys) or method names (`checkLicense`,
`isRooted`), then read the surrounding logic.

## 🛠️ Apktool — Decode & Rebuild

Where JADX decompiles to *readable* (but not directly recompilable) Java, Apktool decodes
an APK to **Smali** (a readable, editable assembly-like format for Dalvik bytecode) that
*can* be edited and rebuilt into a working, re-signed APK.

```bash
apktool d app.apk -o app_decoded/     # decode: APK -> smali + resources
# ... edit app_decoded/smali/.../SomeActivity.smali directly ...
apktool b app_decoded/ -o app_patched.apk   # rebuild
# re-sign, since Android refuses to install an APK whose signature no longer matches
apksigner sign --ks debug.keystore app_patched.apk
```

## 🎣 Dynamic Instrumentation with Frida/objection

Frida injects a JavaScript runtime into a running app, letting you hook and modify
function behavior live, without editing/recompiling the APK at all.

```bash
objection -g com.target.app explore
# drops into an interactive shell inside the running app's process
```
```
android hooking list classes                     # list all loaded classes
android hooking watch class_method com.target.LoginActivity.checkPassword
# logs every call to that method, with its arguments and return value, live
```

## 🔓 Bypassing Client-Side Checks

**Root detection bypass** — apps often refuse to run on a rooted device (a common
security measure, and a common obstacle to testing):
```
objection -g com.target.app explore
android root disable       # objection's built-in root-detection bypass
```

**SSL pinning bypass** — the app hardcodes/pins the expected server certificate, refusing
to trust even a valid CA-signed cert if it doesn't match — this is specifically designed
to defeat exactly the kind of Burp Suite MITM proxy setup from **Day 3**.
```
objection -g com.target.app explore
android sslpinning disable
```
Once pinning is bypassed, Burp can intercept the app's HTTPS traffic just like it did for
a normal browser on Day 3 — same proxy setup, same trust-chain concept, just defeating one
extra layer the app added specifically to stop it.

## 🔗 How This Connects

- JADX decompilation here is the same static-analysis instinct from **Day 11**, applied to
  Java bytecode instead of a native x86/x64 binary.
- SSL-pinning bypass here directly reconnects to **Day 3**'s TLS/PKI trust-chain theory —
  you're now defeating the exact protection built on top of that trust chain.
- These client-bypass techniques are a plausible attack surface if a mobile component ends
  up in scope for the **Day 19-20** capstone.

## 📎 Resources

- `Resources/mobile security.pptx`
- https://github.com/skylot/jadx
- https://github.com/iBotPeaches/Apktool
- https://github.com/frida/frida
- https://github.com/cd80-ctf/InjuredAndroidWriteups

## ✅ Checkpoint / Deliverable

- [ ] SSL-pinning (or equivalent client-side check) bypass demonstrated on a test APK, with Burp successfully intercepting its traffic afterward
- [ ] Decompiled source (JADX) with one interesting finding (hardcoded key/endpoint/logic) documented
