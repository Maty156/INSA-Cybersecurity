# Day 12 — Reverse Engineering: Dynamic Analysis & Debugging

**Date:** Aug 11, 2026
**Week:** Week 3 — Reverse Engineering & Malware Analysis

---

## 🎯 Overview

Where Day 11 read a binary without running it, today runs it under a debugger to watch
behavior live — and covers the tricks binaries use to detect and resist exactly that.

## 🐞 x64dbg — Core Workflow

```
1. File → Open → select the target .exe
2. Set a breakpoint: click the address, press F2 (or right-click → "Toggle Breakpoint")
3. F9 = Run, F7 = Step Into, F8 = Step Over
4. Watch the Registers panel (top-right) and Stack panel (bottom-right) update live
```
- **Step Into (F7):** if the current instruction is a `call`, follow execution into that
  function.
- **Step Over (F8):** execute the `call` as one step, without diving into it — used when
  you trust/don't care about a called function's internals.

**Memory breakpoints** — break when a specific address is read, written, or executed
(rather than a fixed instruction address):
```
Right-click a memory address in the Dump panel → Breakpoint → Memory, Access → Write
```
Useful for questions like "when does the program write to *this* buffer?" without knowing
in advance which instruction does it.

## 🕵️ Anti-Debug Checks

Malware and crackmes alike often check whether they're being debugged, and change
behavior if so (crash, exit silently, or take a fake "success" path to mislead the
analyst).

```c
// classic Windows API check
if (IsDebuggerPresent()) {
    ExitProcess(0);
}
```
**Timing check** — a debugger's breakpoints/stepping slow execution down measurably:
```c
DWORD t1 = GetTickCount();
// ... some operation ...
DWORD t2 = GetTickCount();
if (t2 - t1 > threshold) {
    // "too slow" -> probably being debugged/stepped through
}
```
**Bypass approach:** find the conditional jump right after the check (e.g. `je
not_debugged`), and either patch it (see below) or set the debugger to hide itself
(x64dbg's "ScyllaHide" plugin patches `IsDebuggerPresent` and similar APIs to always
return false).

## 🖥️ Anti-VM Checks

Similar idea, aimed at sandboxes/VMs rather than debuggers — checking for VM-specific
artifacts (hypervisor CPUID bits, VM-tool processes/drivers, low core count/RAM, specific
MAC address OUI prefixes used by VirtualBox/VMware). Malware avoiding execution inside a
VM is trying to dodge automated sandbox analysis — a strong behavioral signal in itself.

## 📦 UPX Unpacking

UPX is a common (legitimate and malicious-use) executable packer — it compresses the
original code and wraps it in a small stub that decompresses it back into memory at
runtime. A packed binary shows almost no readable strings/imports statically (Day 11's
techniques hit a wall) until it's unpacked.

```bash
# Identify a packed binary:
strings packed.exe | grep -i upx
# often literally shows "UPX0", "UPX1" section names

# Unpack directly, if it's standard UPX:
upx -d packed.exe -o unpacked.exe

# If UPX itself won't unpack it (modified header), unpack manually in a debugger:
# 1. Set a breakpoint on a common "unpacking finished" API (e.g. VirtualAlloc/VirtualProtect)
# 2. Run until the stub finishes decompressing into memory
# 3. Dump the now-unpacked process memory (e.g. with Scylla) to a new file for static analysis
```

## ✏️ Binary Patching

Directly editing a binary's bytes to change its behavior — e.g. flipping a conditional
jump so a failed check becomes a passed one.

```
Original:  74 05    (JE  +5   ; jump if equal — "success" branch)
Patched:   EB 05    (JMP +5   ; unconditional jump — always take "success" branch)
```
```bash
# In x64dbg: right-click the instruction -> Binary -> Edit, or Assemble (Space) 
# to type a replacement instruction directly, then File -> Patch File to save changes
```
This is the exact same "override the conditional" concept as Day 12's anti-debug bypass —
find the branch instruction, flip it.

## 🔗 How This Connects

- Anti-debug/anti-VM bypasses learned here are exactly what real malware in **Day 13-14**
  uses to detect a sandbox — recognizing the pattern here means recognizing it again on
  Day 13.
- Binary patching here is a lightweight preview of the C2 config extraction work on
  **Day 15**.
- UPX unpacking is a near-daily first step in real malware triage — expect the exact same
  workflow again on **Day 13**.

## 📎 Resources

- `Resources/RE_Day2_StaticAnalysis.pptx`
- https://p.ost2.fyi/courses/course-v1:OpenSecurityTraining2+Dbg1101_IntroIDA+2024_v1/about

## ✅ Checkpoint / Deliverable

- [ ] Unpacked UPX sample, confirmed by successfully running static analysis (strings/Ghidra) on the output
- [ ] Patched binary that bypasses one anti-debug check, with before/after behavior documented
