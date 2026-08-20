# Day 05 — Computer Architecture & Assembly Intro

**Date:** Jul 31, 2026
**Week:** Week 1 — Foundations

---

## 🎯 Overview

Closing Week 1 one level below C: what actually sits in CPU registers, how a function
call really works at the machine level, x86/x64 assembly basics, and the structure of
compiled executables (PE/ELF). This is the vocabulary every reverse-engineering day in
Week 3 assumes you already have.

## 🧮 CPU Registers

x86-64 has a set of general-purpose registers plus special-purpose ones:

| Register | 64-bit name | Common use |
|----------|-------------|------------|
| Accumulator | RAX | Return values, arithmetic |
| Base | RBX | General storage |
| Counter | RCX | Loop counters |
| Data | RDX | I/O, arithmetic (paired with RAX for large mults) |
| Stack Pointer | RSP | Points to the top of the current stack frame |
| Base Pointer | RBP | Points to the base of the current stack frame |
| Instruction Pointer | RIP | Address of the *next* instruction to execute |

(32-bit versions drop the `R` for `E` — `EAX`, `ESP`, etc.)

## 📚 The Stack & Function Calls

Every function call pushes a new **stack frame** onto the stack. Understanding this frame
is essential for both exploit development and debugging.

```asm
; Function prologue (every function starts like this)
push rbp        ; save caller's base pointer
mov  rbp, rsp    ; establish new base pointer for THIS function's frame
sub  rsp, 0x20   ; reserve 32 bytes of local stack space

; ... function body ...

; Function epilogue
mov  rsp, rbp    ; restore stack pointer
pop  rbp         ; restore caller's base pointer
ret              ; pop return address into RIP, jump there
```

**Calling convention (System V x86-64, Linux):** first 6 integer/pointer arguments go in
`RDI, RSI, RDX, RCX, R8, R9` (not the stack) — the return value comes back in `RAX`.

**Why the return address matters:** it's pushed onto the stack by `call`, sitting right
above the saved RBP. A buffer overflow (introduced in **Day 4**) that overwrites far enough
can overwrite *that return address* — redirecting execution when the function returns.
This is the fundamental primitive behind classic stack-based exploitation.

## 🖥️ x86 vs x64

| | x86 (32-bit) | x64 (64-bit) |
|---|---|---|
| Registers | EAX, EBX... (32-bit) | RAX, RBX... (64-bit), plus R8-R15 |
| Max addressable memory | 4 GB | 16 EB (practically much less) |
| Calling convention | Arguments on the stack | First 6 args in registers |
| Pointer size | 4 bytes | 8 bytes |

## 📖 Reading Disassembly

Two syntax conventions exist for the same instructions:

```
AT&T syntax:   mov %eax, %ebx        (source, dest — GDB default on Linux)
Intel syntax:  mov ebx, eax          (dest, source — Ghidra/IDA default)
```

```asm
mov  eax, [ebp-4]     ; load the local variable at ebp-4 into eax
add  eax, 5            ; eax = eax + 5
cmp  eax, 10            ; compare eax to 10 (sets flags)
jle  loop_start          ; jump if eax <= 10
```

## 📦 PE vs ELF Executable Structure

| | PE (Windows) | ELF (Linux) |
|---|---|---|
| Header | DOS header + PE header | ELF header |
| Code section | `.text` | `.text` |
| Initialized data | `.data` | `.data` |
| Uninitialized data | `.bss` | `.bss` |
| Imports (external functions) | Import Table | Dynamic symbol table (`.dynsym`) |
| Entry point | `AddressOfEntryPoint` | `e_entry` |

```bash
readelf -h ./prog        # ELF header: entry point, architecture, type
readelf -S ./prog        # list all sections
objdump -d ./prog         # full disassembly of executable sections
```

## 🔬 First Disassembly Walkthrough

```bash
objdump -d ./prog | grep -A 20 "<main>:"
```
```
0000000000401136 <main>:
  401136: 55                    push   rbp
  401137: 48 89 e5              mov    rbp,rsp
  40113a: 48 83 ec 10           sub    rsp,0x10
  40113e: c7 45 fc 05 00 00 00  mov    DWORD PTR [rbp-0x4],0x5
  401145: 8b 45 fc              mov    eax,DWORD PTR [rbp-0x4]
  ...
```
Reading this line by line: the prologue (`push rbp`/`mov rbp,rsp`/`sub rsp,0x10`) matches
exactly what was described above — a local stack frame with 16 bytes reserved, then a
local variable (`[rbp-0x4]`) set to `5`.

```bash
gdb ./prog
(gdb) break main
(gdb) run
(gdb) disas          # disassemble the current function
(gdb) stepi           # step one instruction at a time
(gdb) info registers  # dump all register values right now
```

## 🔗 How This Connects

- This is the exact vocabulary — registers, stack frames, PE/ELF sections — reused
  verbatim on **Day 11-12** for real reverse engineering.
- Understanding the prologue/epilogue here is what makes setting a debugger breakpoint on
  Day 12 meaningful instead of arbitrary.
- PE structure knowledge here is reused directly analysing Windows malware on **Day 13-14**.
- The return-address-overwrite concept previewed here is the theoretical basis of the
  buffer overflow introduced in **Day 4**.

## 📎 Resources

- `Resources/computer-architecture-os-assembly.pptx.pdf`
- `Resources/MIT6_S096_IAP13_lec3.pdf`
- https://guyinatuxedo.github.io/01-intro_assembly/assembly/index.html

## ✅ Checkpoint / Deliverable

- [ ] Annotated disassembly of Day 4's compiled program, labelling the stack frame of `main()`
- [ ] GDB session log showing register values stepped through at least 5 instructions
