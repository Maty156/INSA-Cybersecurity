# Day 04 — Programming for Security (Python & C)

**Date:** Jul 30, 2026  
**Week:** Week 1: Foundations  

---

## 🎯 Overview

Python for fast automation/exploit scripting; C for understanding what actually happens in memory — the prerequisite for Week 3's binary work.

## 📚 Topics Covered

- Python: sockets, `subprocess`, argument parsing — writing small automation/exploit-helper scripts
- C: compilation pipeline (`gcc`: preprocess → compile → assemble → link)
- C memory model: stack vs heap, pointers, `malloc`/`free`
- Common C bugs that become vulnerabilities: buffer overflows, dangling pointers
- Very first look at a compiled binary (file type, basic `objdump`/`strings` pass)

## 🛠️ Tools Used

| Tool | Purpose |
|------|---------|
| **gcc** | Compiling C source to an executable |
| **gdb** | First look — inspecting a running/compiled binary |
| **Python venv/pip** | Isolated environments for tool scripts |

## 💻 Key Commands

```bash
gcc -g -o prog prog.c
python3 -c "import socket; ..."
strings ./prog | less
```

## 🔗 How This Connects

- The pointer/stack model taught here is required background for Day 5 (registers/stack) and Day 11-12 (RE).
- A buffer overflow written intentionally today is the same bug class exploited defensively-aware in Week 3.
- Keep every script — Week 2's enumeration days reuse this Python automation pattern constantly.

## 📎 Resources

- Resources/C_Book_2nd.pdf

## ✅ Checkpoint / Deliverable

- [ ] A compiled C program with an intentional memory bug + a Python script that automates a repetitive task.

---
