# Day 11 — Reverse Engineering: Static Analysis

**Date:** Aug 10, 2026  
**Week:** Week 3: Reverse Engineering & Malware Analysis  

---

## 🎯 Overview

Opening Week 3 by reading a compiled binary without running it — the Ghidra/IDA workflow used for the rest of the month.

## 📚 Topics Covered

- Disassembly fundamentals recap (from Day 5) applied to unfamiliar binaries
- Loading a binary into Ghidra / IDA Free and navigating the decompiler view
- PE headers in depth: sections, imports, exports, entry point
- Cross-references (xrefs) — tracing where a function/string is used from
- Solving beginner 'crackme' challenges using only static analysis

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **Ghidra** | Free NSA decompiler/disassembler suite |
| **IDA Free** | Industry-standard disassembler |
| **PEstudio (preview)** | Quick static triage of PE files |

## 💻 Key Commands

```bash
Ghidra: Auto-Analyze on import, then Symbol Tree > Functions
IDA: Shift+F12 for strings window
xrefs: right-click > Show References
```

## 🔗 How This Connects

- This is Day 5's assembly knowledge applied for real, on a binary you didn't write.
- The crackme-solving workflow here (find string → xref → decompile) is reused directly for malware triage on Day 13.
- Static analysis is always the first pass before Day 12's dynamic/debugger work — cheaper and safer.

## 📎 Resources

- Resources/RE_Day1_Foundations.pptx
- https://p.ost2.fyi/.../Dbg1102_IntroGhidra

## ✅ Checkpoint / Deliverable

- [ ] Solved crackme with the recovered flag/condition documented via Ghidra decompiler screenshots.

---
