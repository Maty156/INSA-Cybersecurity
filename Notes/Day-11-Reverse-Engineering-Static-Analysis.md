# Day 11 — Reverse Engineering: Static Analysis

**Date:** Aug 10, 2026
**Week:** Week 3 — Reverse Engineering & Malware Analysis

---

## 🎯 Overview

Opening Week 3 by reading a compiled binary without running it. This is Day 5's assembly
and PE/ELF knowledge applied to an unfamiliar binary for the first time, using the
Ghidra/IDA workflow reused for the rest of the month.

## 🧰 Loading a Binary into Ghidra

```
1. File → New Project → (Non-Shared Project)
2. File → Import File → select the binary
3. Double-click it in the project tree → "Analyze?" → Yes (default analyzers are fine to start)
4. Wait for auto-analysis to finish (progress bar bottom-right)
```
Once analysis finishes, the key panels are:
- **Symbol Tree → Functions** — every function Ghidra identified, by name/address
- **Listing** — raw disassembly, address by address
- **Decompiler** (right panel) — Ghidra's best-effort reconstruction of C-like
  pseudocode from the assembly — usually the fastest way to understand a function's logic

## 🧭 Navigating: Strings → Xrefs → Function

The efficient static-analysis workflow, used constantly this week:

```
1. Window → Defined Strings   → find an interesting string (e.g. "Access Denied", "flag{")
2. Right-click the string → "Show References to Address" (xrefs)
3. Double-click the referencing function → jump straight to the code that uses that string
4. Read the decompiler pane for that function
```
This "start from a string, work backward" pattern is dramatically faster than reading a
binary top-to-bottom, and is the exact same technique used for real malware triage on
**Day 13**.

## 📑 PE Headers in Depth

(Recall the PE/ELF comparison from **Day 5** — going deeper here.)

| Field | Meaning |
|-------|---------|
| DOS Header | Legacy header, mostly just points to the real PE header (`e_lfanew`) |
| PE Header | Machine type, number of sections, timestamp |
| Optional Header | Entry point address, image base, subsystem (GUI/console) |
| Section Table | List of sections (`.text`, `.data`, `.rdata`) with size/permissions |
| Import Table | External DLLs and functions this binary calls (`kernel32.dll!CreateFileA`) |
| Export Table | Functions this binary makes available to others (mainly DLLs) |

**Imports are one of the fastest ways to guess behavior before reading any code** — a
binary importing `InternetOpenA`/`send` talks to the network; one importing
`CryptEncrypt` does something with encryption; one importing `RegSetValueEx` touches the
registry (persistence — foreshadowing **Day 14**).

## 🔗 Cross-References (Xrefs)

Xrefs answer "where else is this used?" — for a function, a global variable, or a string.
In Ghidra: right-click any symbol → **Show References To**. This turns a flat disassembly
into a navigable graph, which is how real analysts move through large binaries instead of
reading linearly.

## 🧩 Solving a Beginner Crackme (Static-Only)

A typical simple crackme checks input against a hardcoded value:

```c
// Ghidra decompiler output (cleaned up)
int check_password(char *input) {
    if (strcmp(input, "S3cr3t_Fl4g!") == 0) {
        return 1;   // success
    }
    return 0;       // failure
}
```
Workflow: find the string `"S3cr3t_Fl4g!"` in Defined Strings → xref to `check_password`
→ read the decompiler → the flag is right there, without ever running the binary. This is
"solving via static analysis" — the whole point of today's checkpoint.

## 🔍 PEstudio — Quick Static Triage Preview

Before deep-diving in Ghidra, PEstudio gives a fast red-flag pass on a PE file: imports,
sections, embedded strings, and a basic "indicators" score — useful as a triage step
before committing to a full Ghidra session (this becomes central on **Day 13**).

## 🔗 How This Connects

- This is **Day 5**'s assembly/PE knowledge applied for real, on a binary you didn't write.
- The "string → xref → decompile" workflow here is reused directly for malware triage on
  **Day 13**.
- Static analysis is always the first, safer pass before **Day 12**'s dynamic/debugger
  work — cheaper and lower-risk (nothing actually executes).
- Import-table reading here (spotting `CryptEncrypt`, `InternetOpenA`, etc.) is exactly
  how malware capability is guessed before detonating anything on Day 13.

## 📎 Resources

- `Resources/RE_Day1_Foundations.pptx`
- https://p.ost2.fyi/courses/course-v1:OpenSecurityTraining2+Dbg1102_IntroGhidra+2024_v2/about

## ✅ Checkpoint / Deliverable

- [ ] Solved crackme with the recovered flag/condition, documented via Ghidra decompiler screenshots
- [ ] Import-table analysis of one unfamiliar binary, with a guessed capability list before running it
