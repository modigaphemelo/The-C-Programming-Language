# 01: History — Why C Exists, Where It's Used

---

## The Problem That Created C

By the late 1960s, operating systems were written in assembly. Each machine had its own instruction set. Porting an OS to new hardware meant rewriting thousands of lines of assembly from scratch. This was unsustainable.

The Multics project (1965–1969) attempted to solve this with PL/I, a high-level language. PL/I was complex. The compiler generated slow code. The system collapsed under its own weight. Ken Thompson and Dennis Ritchie, Bell Labs researchers who had worked on Multics, retreated to a smaller vision: Unix.

They needed:

- A language for **systems programming** — direct memory access, bit manipulation, minimal runtime
- That generated **efficient machine code** — competing with assembly
- That was **portable** across different machines
- With a **simple compiler** — limited hardware resources

Assembly wasn't portable. Fortran couldn't touch memory directly. PL/I was too heavy.

## The Birth: B, Then C

Thompson created **B** in 1969, a stripped-down version of BCPL. B was typeless—everything was a word. It worked on the PDP-7 but failed on the PDP-11 when they added byte addressing. B couldn't express bytes.

Ritchie extended B with types, calling it **C** in 1972. The new language gave them:

- `char` and `int` types matching hardware
- Pointers for direct memory access
- Compiled to efficient code because abstractions mapped directly to machine operations
- Minimal runtime — no garbage collector, no virtual machine

By 1973, Unix was rewritten in C. This was the turning point: an operating system written in a high-level language, performant enough to compete with assembly. Unix could now be ported to new machines by recompiling, not rewriting.

## Why C Spread

C escaped Bell Labs for three reasons:

1. **The PDP-11 was everywhere.** Universities and research labs had them. C and Unix came as a package.
2. **The compiler was simple and retargetable.** Porting C to a new architecture took months, not years.
3. **K&R (1978).** Kernighan and Ritchie wrote *The C Programming Language*. It was small, precise, and became the de facto standard until ANSI formalized it in 1989.

By the 1980s, C was the lingua franca of systems programming. If you were writing an operating system, a database, a compiler, or a networking stack, you wrote it in C. There was no serious alternative that gave both control and portability.

## Where C Lives Today

C survives in domains where control, predictability, and minimal overhead are non-negotiable.

### Operating Systems Kernels
- Linux (~95% C)
- Windows kernel (C and some C++)
- BSD kernels
- Every RTOS you've heard of

### Embedded Systems
- Microcontrollers with KB of RAM — no room for C++ exceptions or Rust's runtime
- Firmware for devices that must not fail (medical, automotive, aerospace)
- Bootloaders, BIOS/UEFI

### Language Implementations
- Python interpreter (CPython) — C
- Ruby (MRI) — C
- PHP, Lua, SQLite, Git, Redis — all C
- Java's JVM — C/C++

### Performance-Critical Libraries
- Numerical computing (BLAS, LAPACK)
- Cryptography (OpenSSL)
- Compression (zlib)
- Database engines (SQLite, core of MySQL/PostgreSQL)

### When You Must Talk to Hardware
- Device drivers
- Firmware
- Kernel modules
- Any place where memory-mapped I/O, interrupts, and precise timing matter

## Why C Refuses to Die

Languages come and go. C persists because:

- **It maps to the machine.** The C abstract machine is close to real hardware. You can predict what the compiler will generate.
- **It has no hidden costs.** No garbage collection pauses, no reference counting overhead, no virtual dispatch. You pay for exactly what you write.
- **It can be called from anywhere.** Every language provides a way to call C. C is the lowest common denominator.
- **The ecosystem is vast.** Fifty years of libraries, tools, and expertise.

The tradeoff: no safety nets. Buffer overflows, use-after-free, uninitialized variables — C trusts you completely. This is why it built the modern world, and why modern languages (Rust, Go, Zig) are trying to replace it in specific domains.

## What This Means for You

Learning C is learning how the machine actually works. Every abstraction you've ever relied on in other languages is implemented in C. When you understand C, you understand the foundation.

Most programmers never touch C. They build on abstractions they don't understand. When something breaks, they have no idea what is happening beneath the surface.

You will not have this problem.

## Cross-References

- **Next:** [02 — Structure of a C Program](02-program-structure.md)
- The Unix operating system, written in C, is the direct ancestor of Linux, macOS, and BSD
- Memory safety concerns in C directly motivated Rust's ownership system
- Embedded C constraints shape how microcontrollers are programmed

## References

- Ritchie, D. M. (1993). *The Development of the C Language*
- Kernighan, B. W., & Ritchie, D. M. (1978). *The C Programming Language*
- Salus, P. H. (1994). *A Quarter Century of UNIX*