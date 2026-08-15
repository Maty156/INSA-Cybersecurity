# Day 12 — Reverse Engineering: Dynamic Analysis & Debugging

**Date:** Aug 11, 2026  
**Week:** Week 3: Reverse Engineering & Malware Analysis  

---

## 🎯 Overview

Running the binary under a debugger to watch behaviour live, plus common tricks binaries use to make that harder.

## 📚 Topics Covered

- x64dbg setup and core workflow (breakpoints, stepping, register/stack view)
- Memory breakpoints (break on read/write/execute of an address)
- Anti-debug checks (`IsDebuggerPresent`, timing checks) and how to bypass them
- Anti-VM checks and why malware/crackmes use them
- UPX unpacking — identifying and unpacking a packed binary
- Binary patching — flipping a conditional jump to change program behaviour

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **x64dbg** | Windows user-mode debugger |
| **UPX** | Packer, used here in reverse to unpack |
| **HxD / hex editor** | Direct binary patching |

## 💻 Key Commands

```bash
upx -d packed.exe
x64dbg: bp <address>; run; step into (F7)
patch: JE -> JMP (0x74 -> 0xEB)
```

## 🔗 How This Connects

- Anti-debug/anti-VM bypasses here are exactly what real malware in Day 13-14 uses to detect a sandbox.
- Binary patching is a lightweight preview of the C2 config extraction work on Day 15.
- UPX unpacking is a near-daily first step in real malware triage — expect it again on Day 13.

## 📎 Resources

- Resources/RE_Day2_StaticAnalysis.pptx
- https://p.ost2.fyi/.../Dbg1101_IntroIDA

## ✅ Checkpoint / Deliverable

- [ ] Unpacked UPX sample + a patched binary that bypasses one anti-debug check.

---
