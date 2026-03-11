# The C Programming Language

*The language that built the modern world. No runtime, no safety nets, no excuses.*

**Target:** January 14, 2027  
**Started:** March 11, 2026  
**Status:** Active

---

## Overview

This repository documents the path to C mastery. Each section contains:
- **Notes/** — Detailed explanations of concepts
- **Projects/** — Applied exercises to cement understanding
- **Code/** — Example implementations and experiments

The voice is professional, precise, and assumes an intelligent reader. No analogies. No hand-holding. Just C, as it exists in the kernel, the embedded system, and the high-performance server.

---

## Mastery Checklist

### Part 01: Foundations — The Absolute Basics
- [ ] [History — Why C exists, where it's used](/01-foundations/notes/01-history.md)
- [ ] [Structure of a C Program — `main()`, headers](/01-foundations/notes/02-program-structure.md)
- [ ] [Compilation — Preprocessor, compiler, linker, assembler](/01-foundations/notes/03-compilation.md)
- [ ] [Basic Types — `int`, `char`, `float`, `double`](/01-foundations/notes/04-basic-types.md)
- [ ] [Variables — Declaration, initialization, scope](/01-foundations/notes/05-variables.md)
- [ ] [Constants — `const`, `#define`, enums](/01-foundations/notes/06-constants.md)
- [ ] [Operators — Arithmetic, relational, logical, bitwise](/01-foundations/notes/07-operators.md)
- [ ] [Control Flow — `if`, `else`, `switch`](/01-foundations/notes/08-control-flow.md)
- [ ] [Loops — `for`, `while`, `do-while`](/01-foundations/notes/09-loops.md)

**Project:** [Hello World and Variations](/01-foundations/projects/01-hello-world/)

---

### Part 02: Functions — The Building Blocks
- [ ] [Function Syntax — Return type, parameters, body](/02-functions/notes/10-function-syntax.md)
- [ ] [Pass by Value — Everything is a copy](/02-functions/notes/11-pass-by-value.md)
- [ ] [Function Prototypes — Declaration vs definition](/02-functions/notes/12-prototypes.md)
- [ ] [Scope and Lifetime — Local, global, static](/02-functions/notes/13-scope.md)
- [ ] [Recursion — Base cases, stack depth](/02-functions/notes/14-recursion.md)
- [ ] [Inline Functions — Hinting the compiler](/02-functions/notes/15-inline.md)
- [ ] [Variadic Functions — `stdarg.h`, `printf`-like](/02-functions/notes/16-variadic.md)

**Project:** [Function Library](/02-functions/projects/02-function-library/)

---

### Part 03: Arrays and Strings
- [ ] [Arrays — Declaration, initialization, access](/03-arrays-strings/notes/17-arrays.md)
- [ ] [Multidimensional Arrays — Rows and columns](/03-arrays-strings/notes/18-multidimensional.md)
- [ ] [Strings — Character arrays, null terminator](/03-arrays-strings/notes/19-strings.md)
- [ ] [String Functions — `string.h`, `strlen`, `strcpy`, etc.](/03-arrays-strings/notes/20-string-functions.md)
- [ ] [Array and Pointer Relationship](/03-arrays-strings/notes/21-array-pointer.md)
- [ ] [Array of Strings — `char *argv[]`](/03-arrays-strings/notes/22-array-of-strings.md)

**Project:** [String Manipulation Library](/03-arrays-strings/projects/03-string-library/)

---

### Part 04: Pointers — The Heart of C
- [ ] [What is a Pointer? — Memory addresses](/04-pointers/notes/23-pointer-basics.md)
- [ ] [Pointer Operators — `&` (address), `*` (dereference)](/04-pointers/notes/24-pointer-operators.md)
- [ ] [Pointer Arithmetic — Moving through memory](/04-pointers/notes/25-pointer-arithmetic.md)
- [ ] [Null Pointers — `NULL`, nullptr, dangers](/04-pointers/notes/26-null.md)
- [ ] [Pointers and Functions — Pass by pointer](/04-pointers/notes/27-pointer-parameters.md)
- [ ] [Pointers to Pointers — `**`](/04-pointers/notes/28-pointer-to-pointer.md)
- [ ] [Function Pointers — Callbacks, jump tables](/04-pointers/notes/29-function-pointers.md)
- [ ] [Void Pointers — `void*`, generic pointers](/04-pointers/notes/30-void-pointers.md)

**Project:** [Pointer Explorer](/04-pointers/projects/04-pointer-explorer/)

---

### Part 05: Memory Management
- [ ] [Stack vs Heap — Lifetime and size differences](/05-memory/notes/31-stack-heap.md)
- [ ] [Static Allocation — Compile-time fixed size](/05-memory/notes/32-static-allocation.md)
- [ ] [Dynamic Allocation — `malloc`, `calloc`, `realloc`](/05-memory/notes/33-dynamic-allocation.md)
- [ ] [Freeing Memory — `free`, dangling pointers](/05-memory/notes/34-free.md)
- [ ] [Memory Leaks — Detection and prevention](/05-memory/notes/35-memory-leaks.md)
- [ ] [Memory Layout — Text, data, bss, heap, stack](/05-memory/notes/36-memory-layout.md)
- [ ] [Alignment and Padding — Structure packing](/05-memory/notes/37-alignment.md)

**Project:** [Dynamic Array Implementation](/05-memory/projects/05-dynamic-array/)

---

### Part 06: Structures and Unions
- [ ] [Structures — `struct`, grouping data](/06-structures-unions/notes/38-structures.md)
- [ ] [Structure Padding — Why sizeof lies](/06-structures-unions/notes/39-struct-padding.md)
- [ ] [Pointers to Structures — `->` operator](/06-structures-unions/notes/40-struct-pointers.md)
- [ ] [Nested Structures](/06-structures-unions/notes/41-nested-structs.md)
- [ ] [Unions — Overlapping memory](/06-structures-unions/notes/42-unions.md)
- [ ] [Bit Fields — Packing flags](/06-structures-unions/notes/43-bit-fields.md)
- [ ] [`typedef` — Creating type aliases](/06-structures-unions/notes/44-typedef.md)

**Project:** [Data Structure Library](/06-structures-unions/projects/06-data-structures/)

---

### Part 07: Input/Output
- [ ] [Standard I/O — `stdio.h`](/07-io/notes/45-stdio.md)
- [ ] [Formatted Output — `printf` family](/07-io/notes/46-printf.md)
- [ ] [Formatted Input — `scanf` family and pitfalls](/07-io/notes/47-scanf.md)
- [ ] [File I/O — `fopen`, `fclose`, `fread`, `fwrite`](/07-io/notes/48-file-io.md)
- [ ] [Line-by-line Reading — `fgets`, `getline`](/07-io/notes/49-line-reading.md)
- [ ] [Binary vs Text Files](/07-io/notes/50-binary-text.md)
- [ ] [Error Handling — `ferror`, `perror`](/07-io/notes/51-file-errors.md)

**Project:** [File Processor](/07-io/projects/07-file-processor/)

---

### Part 08: Preprocessor
- [ ] [Preprocessor Directives — `#include`, `#define`](/08-preprocessor/notes/52-preprocessor.md)
- [ ] [Macros — Object-like, function-like](/08-preprocessor/notes/53-macros.md)
- [ ] [Conditional Compilation — `#ifdef`, `#ifndef`, `#if`](/08-preprocessor/notes/54-conditional.md)
- [ ] [Macro Pitfalls — Double evaluation, parentheses](/08-preprocessor/notes/55-macro-pitfalls.md)
- [ ] [Stringification — `#` operator](/08-preprocessor/notes/56-stringification.md)
- [ ] [Concatenation — `##` operator](/08-preprocessor/notes/57-concatenation.md)
- [ ] [Predefined Macros — `__FILE__`, `__LINE__`, etc.](/08-preprocessor/notes/58-predefined.md)

**Project:** [Debug Macros](/08-preprocessor/projects/08-debug-macros/)

---

### Part 09: Advanced Pointers
- [ ] [Complex Declarations — Reading right-to-left](/09-advanced-pointers/notes/59-complex-declarations.md)
- [ ] [Pointer to Array vs Array of Pointers](/09-advanced-pointers/notes/60-pointer-array-confusion.md)
- [ ] [Pointer to Function Returning Pointer...](/09-advanced-pointers/notes/61-extreme-pointers.md)
- [ ] [Restrict Keyword — Aliasing hints](/09-advanced-pointers/notes/62-restrict.md)
- [ ] [Volatile — Preventing optimizations](/09-advanced-pointers/notes/63-volatile.md)

**Project:** [Pointer Puzzle Solver](/09-advanced-pointers/projects/09-pointer-puzzles/)

---

### Part 10: Libraries and Linking
- [ ] [Static Libraries — `.a` files, `ar`](/10-libraries/notes/64-static-libraries.md)
- [ ] [Shared Libraries — `.so`, `.dll`, `.dylib`](/10-libraries/notes/65-shared-libraries.md)
- [ ] [Dynamic Loading — `dlopen`, `dlsym`](/10-libraries/notes/66-dynamic-loading.md)
- [ ] [Header Guards — Preventing multiple inclusion](/10-libraries/notes/67-header-guards.md)
- [ ] [Linker Scripts — Advanced control](/10-libraries/notes/68-linker-scripts.md)

**Project:** [Library Creator](/10-libraries/projects/10-library-creator/)

---

### Part 11: System Programming
- [ ] [Processes — `fork`, `exec`, `wait`](/11-system/notes/69-processes.md)
- [ ] [Signals — `signal`, `kill`, handling](/11-system/notes/70-signals.md)
- [ ] [Pipes — Inter-process communication](/11-system/notes/71-pipes.md)
- [ ] [Sockets — Network programming basics](/11-system/notes/72-sockets.md)
- [ ] [File Descriptors — Low-level I/O](/11-system/notes/73-file-descriptors.md)
- [ ] [Memory Mapping — `mmap`](/11-system/notes/74-mmap.md)
- [ ] [Threads — POSIX threads (pthreads)](/11-system/notes/75-threads.md)
- [ ] [Synchronization — Mutexes, condition variables](/11-system/notes/76-synchronization.md)

**Project:** [Mini Web Server](/11-system/projects/11-mini-server/)

---

### Part 12: Debugging and Tools
- [ ] [GDB — The GNU Debugger](/12-tools/notes/77-gdb.md)
- [ ] [Valgrind — Memory error detection](/12-tools/notes/78-valgrind.md)
- [ ] [Address Sanitizer — Modern memory checking](/12-tools/notes/79-asan.md)
- [ ] [Make — Build automation](/12-tools/notes/80-make.md)
- [ ] [CMake — Cross-platform builds](/12-tools/notes/81-cmake.md)
- [ ] [Static Analysis — `lint`, `clang-tidy`](/12-tools/notes/82-static-analysis.md)
- [ ] [Profiling — `gprof`, `perf`](/12-tools/notes/83-profiling.md)

**Project:** [Debugging Exercise Suite](/12-tools/projects/12-debugging-exercises/)

---

### Part 13: Undefined Behavior
- [ ] [What is Undefined Behavior?](/13-ub/notes/84-ub.md)
- [ ] [Common UB — Overflow, null dereference, etc.](/13-ub/notes/85-ub-common.md)
- [ ] [UB and Optimization — Compiler assumptions](/13-ub/notes/86-ub-optimization.md)
- [ ] [Implementation-Defined Behavior](/13-ub/notes/87-implementation-defined.md)
- [ ] [Unspecified Behavior](/13-ub/notes/88-unspecified.md)

**Project:** [UB Hunter](/13-ub/projects/13-ub-hunter/)

---

### Part 14: C Standards
- [ ] [K&R C — The original](/14-standards/notes/89-kr.md)
- [ ] [ANSI C (C89/C90) — Standardization](/14-standards/notes/90-ansi.md)
- [ ] [C99 — Designated initializers, inline, comments](/14-standards/notes/91-c99.md)
- [ ] [C11 — Threads, atomics, `_Generic`](/14-standards/notes/92-c11.md)
- [ ] [C17/C18 — Bug fixes](/14-standards/notes/93-c17.md)
- [ ] [C2x — The future](/14-standards/notes/94-c2x.md)

**Project:** [Standards Explorer](/14-standards/projects/14-standards-explorer/)

---

### Part 15: Real-World C
- [ ] [Reading Others' Code — Linux kernel, git, redis](/15-real-world/notes/95-reading-code.md)
- [ ] [Coding Standards — Style, consistency](/15-real-world/notes/96-coding-standards.md)
- [ ] [Portability — Writing cross-platform C](/15-real-world/notes/97-portability.md)
- [ ] [Embedded C — Constraints and patterns](/15-real-world/notes/98-embedded.md)
- [ ] [Security — Buffer overflows, format string attacks](/15-real-world/notes/99-security.md)
- [ ] [Interfacing with Other Languages — FFI, Python/C API](/15-real-world/notes/100-ffi.md)

**Project:** [Contribute to Open Source C Project](/15-real-world/projects/15-open-source/)

---

## Capstone Project

After completing all sections, build one of:

- A simple garbage collector
- A basic HTTP server
- A memory allocator (like malloc)
- An interpreter for a tiny language
- A game (tetris) with a simple graphics library

**Repository:** [/capstone/](/capstone/)

---

## Directory Structure

```

/
├── README.md (this file)
├── 01-foundations/
│   ├── notes/
│   └── projects/
├── 02-functions/
│   ├── notes/
│   └── projects/
├── 03-arrays-strings/
│   ├── notes/
│   └── projects/
├── 04-pointers/
│   ├── notes/
│   └── projects/
├── 05-memory/
│   ├── notes/
│   └── projects/
├── 06-structures-unions/
│   ├── notes/
│   └── projects/
├── 07-io/
│   ├── notes/
│   └── projects/
├── 08-preprocessor/
│   ├── notes/
│   └── projects/
├── 09-advanced-pointers/
│   ├── notes/
│   └── projects/
├── 10-libraries/
│   ├── notes/
│   └── projects/
├── 11-system/
│   ├── notes/
│   └── projects/
├── 12-tools/
│   ├── notes/
│   └── projects/
├── 13-ub/
│   ├── notes/
│   └── projects/
├── 14-standards/
│   ├── notes/
│   └── projects/
├── 15-real-world/
│   ├── notes/
│   └── projects/
└── capstone/

```

---

## Progress Tracking

| Part | Status | Completed |
|------|--------|-----------|
| 01: Foundations | ⏳ In Progress | 0/9 |
| 02: Functions | ⏳ Not Started | 0/7 |
| 03: Arrays and Strings | ⏳ Not Started | 0/6 |
| 04: Pointers | ⏳ Not Started | 0/8 |
| 05: Memory Management | ⏳ Not Started | 0/7 |
| 06: Structures and Unions | ⏳ Not Started | 0/7 |
| 07: Input/Output | ⏳ Not Started | 0/7 |
| 08: Preprocessor | ⏳ Not Started | 0/7 |
| 09: Advanced Pointers | ⏳ Not Started | 0/5 |
| 10: Libraries and Linking | ⏳ Not Started | 0/5 |
| 11: System Programming | ⏳ Not Started | 0/8 |
| 12: Debugging and Tools | ⏳ Not Started | 0/7 |
| 13: Undefined Behavior | ⏳ Not Started | 0/5 |
| 14: C Standards | ⏳ Not Started | 0/6 |
| 15: Real-World C | ⏳ Not Started | 0/6 |

**Overall:** 0/100 topics (0%)

Last updated: March 11, 2026  
Target completion: January 14, 2027

---

## Resources

- [The C Programming Language (K&R)](https://en.wikipedia.org/wiki/The_C_Programming_Language)
- [Linux man pages](https://man7.org/linux/man-pages/)
