# Day 05 — Computer Architecture & Assembly Intro

**Date:** Jul 31, 2026  
**Week:** Week 1: Foundations  

---

## 🎯 Overview

Closing out Week 1 one level below C: CPU registers, the stack/heap at the machine level, x86/x64 assembly, and executable file formats.

## 📚 Topics Covered

- CPU registers: general purpose (EAX/RAX...), instruction pointer, stack pointer/base pointer
- Stack frame mechanics: `push`/`pop`, calling conventions, function prologue/epilogue
- x86 vs x64 instruction set differences
- PE (Windows) vs ELF (Linux) executable structure — headers, sections (.text/.data/.bss)
- Reading disassembly for the first time (AT&T vs Intel syntax)

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **objdump** | Disassembling a compiled binary |
| **readelf / file** | Inspecting ELF headers and sections |
| **gdb (disas)** | Stepping through assembly instructions live |

## 💻 Key Commands

```bash
objdump -d ./prog
readelf -h ./prog
gdb: disas main
```

## 🔗 How This Connects

- This is the exact vocabulary (registers, stack frames, PE/ELF) reused verbatim in Day 11-12 static/dynamic RE.
- Understanding the function prologue/epilogue is what makes a debugger breakpoint make sense later.
- PE structure knowledge here is reused directly analysing Windows malware in Day 13-14.

## 📎 Resources

- Resources/computer-architecture-os-assembly.pptx.pdf
- Resources/MIT6_S096_IAP13_lec3.pdf
- https://guyinatuxedo.github.io/01-intro_assembly/assembly/index.html

## ✅ Checkpoint / Deliverable

- [ ] Annotated disassembly of Day 4's compiled program, labelling the stack frame of `main()`.

---
