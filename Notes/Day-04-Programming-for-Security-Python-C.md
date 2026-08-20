# Day 04 — Programming for Security (Python & C)

**Date:** Jul 30, 2026
**Week:** Week 1 — Foundations

---

## 🎯 Overview

Python for fast, disposable automation/exploit-helper scripts; C for understanding what
actually happens in memory when a program runs — the required background for Week 3's
binary/RE work.

## 🐍 Python for Security Automation

**Sockets** — talking directly to a service over TCP, the way `nc`/Nmap do under the hood:

```python
import socket

def banner_grab(host, port):
    s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    s.settimeout(3)
    s.connect((host, port))
    banner = s.recv(1024).decode(errors="ignore")
    s.close()
    return banner

print(banner_grab("192.168.1.10", 22))
# SSH-2.0-OpenSSH_8.9p1 Ubuntu-3ubuntu0.4
```

**subprocess** — calling other tools (Nmap, etc.) from inside a Python script and capturing
their output:

```python
import subprocess

result = subprocess.run(["nmap", "-sV", "192.168.1.10"],
                         capture_output=True, text=True)
print(result.stdout)
```

**argparse** — turning a one-off script into a reusable CLI tool:

```python
import argparse

parser = argparse.ArgumentParser(description="Simple port scanner")
parser.add_argument("host")
parser.add_argument("-p", "--ports", default="1-1024")
args = parser.parse_args()
print(f"Scanning {args.host} on ports {args.ports}")
```

## 🔧 C — Compilation Pipeline

Unlike Python, C is compiled ahead of time through 4 distinct stages:

```
prog.c → [preprocess] → prog.i → [compile] → prog.s (assembly)
       → [assemble]   → prog.o (machine code, object file)
       → [link]       → prog (final executable)
```

```bash
gcc -E prog.c -o prog.i     # preprocessing only (expand #include, #define)
gcc -S prog.c -o prog.s     # compile to assembly (readable!)
gcc -c prog.c -o prog.o     # assemble to object code
gcc prog.o -o prog          # link into a final executable
gcc -g -o prog prog.c       # all-in-one, with debug symbols (-g) for gdb later
```
Seeing `prog.s` (the `-S` output) is the direct bridge into **Day 5**'s assembly reading —
it's the exact human-readable assembly the CPU's instructions come from.

## 🧠 C Memory Model — Stack vs Heap

```c
#include <stdlib.h>

void example() {
    int local_var = 5;              // STACK: fixed size, freed automatically on return
    int *heap_var = malloc(sizeof(int)); // HEAP: manually managed
    *heap_var = 10;
    free(heap_var);                 // must free manually, or it's a memory leak
}
```

| | Stack | Heap |
|---|-------|------|
| Managed by | Compiler (automatic) | Programmer (`malloc`/`free`) |
| Speed | Fast | Slower |
| Size | Fixed, limited (stack overflow if exceeded) | Large, flexible |
| Lifetime | Until function returns | Until explicitly freed |

**Pointers** — a variable that holds a memory address:

```c
int x = 42;
int *ptr = &x;      // ptr holds the ADDRESS of x
printf("%d\n", *ptr); // dereference: follow the pointer -> prints 42
*ptr = 100;           // writing through the pointer changes x itself
```

## 🐞 Common C Bugs That Become Vulnerabilities

**Buffer overflow** — writing past the end of a fixed-size buffer, overwriting adjacent
stack memory (including, potentially, the return address):

```c
void vulnerable(char *input) {
    char buffer[16];
    strcpy(buffer, input);   // NO bounds check — if input > 16 bytes, overflow!
}
```
This exact bug class, understood conceptually here, is what **Day 5**'s stack-frame
diagrams make visually concrete, and what real-world exploit development targets.

**Dangling pointer / use-after-free** — using a pointer after its memory has been freed:

```c
int *p = malloc(sizeof(int));
free(p);
*p = 5;   // undefined behavior — p still points at freed memory
```

## 🔍 First Look at a Compiled Binary

```bash
file ./prog
# prog: ELF 64-bit LSB pie executable, x86-64 ...

strings ./prog | less
# dumps every printable string embedded in the binary — often reveals
# hardcoded paths, debug messages, even credentials in poorly-written programs
```

`file` and `strings` are the two fastest, lowest-effort static-analysis commands there
are — always run them first, before reaching for Ghidra (**Day 11**).

## 🔗 How This Connects

- The stack/heap/pointer model here is required background for **Day 5** (registers/stack
  frames) and **Day 11-12** (reverse engineering).
- The buffer overflow written intentionally today is the same bug class analysed
  defensively in Week 3.
- `strings`/`file` here become the very first two commands run against any unknown binary
  or malware sample for the rest of the program.
- The Python socket/subprocess patterns here get reused constantly for Week 2's
  enumeration scripts.

## 📎 Resources

- `Resources/C_Book_2nd.pdf`

## ✅ Checkpoint / Deliverable

- [ ] A compiled C program containing one intentional buffer-overflow bug (documented, not necessarily exploited yet)
- [ ] A Python script that automates a repetitive task (e.g. banner-grabbing a list of hosts)
