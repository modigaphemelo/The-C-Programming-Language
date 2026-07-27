# 03: Compilation — Preprocessor, Compiler, Assembler, Linker

---

## The Four Stages

C source code does not become an executable in one step. The transformation happens in four distinct stages, each with a specific purpose. Understanding them is not academic—it is essential for debugging, performance optimization, and understanding why certain errors occur.

```
Source (.c) ──► Preprocessor ──► Compiler ──► Assembler ──► Linker ──► Executable
```

Each stage transforms the code into a progressively lower-level representation:

| Stage | Input | Output | Tool |
|---|---|---|---|
| Preprocessor | Source file | Expanded source | `cpp` (preprocessor) |
| Compiler | Expanded source | Assembly code | `cc1` (compiler) |
| Assembler | Assembly code | Object file | `as` (assembler) |
| Linker | Object files + libraries | Executable | `ld` (linker) |

---

## 1. Preprocessor

The preprocessor handles directives that begin with `#`. It runs before the compiler sees the code.

**What it does:**

- `#include` — Textually inserts the contents of the specified file
- `#define` — Macro substitution (text replacement)
- `#ifdef`, `#ifndef`, `#if` — Conditional compilation
- `#undef` — Removes a macro definition
- `#pragma` — Implementation-specific instructions

**Example:**

```c
#define PI 3.14159
#define SQUARE(x) ((x) * (x))

#ifdef DEBUG
    printf("Debug: value is %d\n", value);
#endif
```

After preprocessing, `PI` becomes `3.14159`, `SQUARE(5)` becomes `((5) * (5))`, and the `printf` is either included or removed based on whether `DEBUG` is defined.

**View the preprocessed output:**

```bash
gcc -E source.c
```

This outputs the expanded source without compiling. Useful for debugging macro expansions and include paths.

---

## 2. Compiler

The compiler takes the preprocessed source and generates assembly code for the target architecture.

**What it does:**

- Lexical analysis — tokenizes the source
- Syntax analysis — parses into an abstract syntax tree (AST)
- Semantic analysis — type checking, symbol resolution
- Optimization — constant folding, dead code elimination, loop unrolling, etc.
- Code generation — emits assembly instructions for the target CPU

**View the assembly output:**

```bash
gcc -S source.c
```

This outputs `source.s` containing assembly code. Reading assembly is not required for most C programming, but it can be useful for understanding performance bottlenecks.

**Common compiler flags:**

| Flag | Purpose |
|---|---|
| `-O0` | No optimization (default, fastest compilation, slowest execution) |
| `-O1` | Basic optimization (size and speed) |
| `-O2` | Moderate optimization (recommended for most code) |
| `-O3` | Aggressive optimization (may increase binary size) |
| `-Os` | Optimize for size (embedded systems) |
| `-Wall` | Enable all common warnings |
| `-Wextra` | Enable extra warnings |
| `-Werror` | Treat warnings as errors |
| `-std=c99` | Use the C99 standard |
| `-std=c11` | Use the C11 standard |
| `-std=c17` | Use the C17 standard |

---

## 3. Assembler

The assembler converts assembly language into machine code, producing an object file (`.o` or `.obj`).

**What it does:**

- Translates mnemonics (`mov`, `add`, `push`) to machine code bytes
- Resolves addresses within the object file
- Creates symbol tables for unresolved references

**View the object file's contents:**

```bash
objdump -d source.o        # Disassemble
nm source.o                 # List symbols
readelf -h source.o         # ELF header (Linux)
```

**Object file structure:**

An object file contains:

- Code section (`.text`) — executable instructions
- Data sections (`.data`, `.rodata`, `.bss`) — initialized, read-only, and zero-initialized data
- Symbol table — list of defined and undefined symbols
- Relocation information — addresses that must be fixed by the linker
- Debug information — line numbers, variable names (if `-g` used)

**File formats:**

- ELF — Linux, Unix, BSD
- Mach-O — macOS, iOS
- COFF/PE — Windows

---

## 4. Linker

The linker combines object files and libraries into a single executable.

**What it does:**

- Symbol resolution — matches references to definitions across translation units
- Relocation — assigns final memory addresses to code and data
- Library linking — resolves symbols from static and shared libraries
- Generates the final executable (ELF, Mach-O, PE)

**Symbol resolution:**

When the linker sees an undefined symbol (like `printf`), it searches object files and libraries to find its definition. If not found, it produces an error: `undefined reference`.

**View linked symbols:**

```bash
objdump -T myprogram      # Dynamic symbols
nm myprogram              # All symbols
ldd myprogram             # Shared library dependencies (Linux)
otool -L myprogram        # Shared library dependencies (macOS)
```

**Static vs dynamic linking:**

| Type | How It Works | Binary Size | Load Time |
|---|---|---|---|
| Static | Library code is copied into the executable | Larger | Faster |
| Dynamic | Library code is loaded at runtime | Smaller | Slower (runtime resolution) |

**Static linking:**

```bash
gcc -static source.c -o program
```

**Dynamic linking (default):**

```bash
gcc source.c -o program
```

The runtime linker (`ld.so` on Linux) resolves dynamic symbols when the program starts.

---

## The Complete Flow

```bash
# Step by step (manual)

# 1. Preprocess
gcc -E source.c -o source.i

# 2. Compile to assembly
gcc -S source.i -o source.s

# 3. Assemble to object
as source.s -o source.o

# 4. Link to executable
gcc source.o -o program

# All at once (typical)
gcc source.c -o program
```

When you run `gcc source.c -o program`, all four stages happen automatically.

---

## Common Errors by Stage

**Preprocessor errors:**

```
source.c:5:10: fatal error: 'missing.h' file not found
```

The include path is wrong, or the file doesn't exist.

**Compiler errors:**

```
source.c:12:15: error: expected ';' after expression
```

Syntax error in the source.

**Compiler warnings (not errors, but important):**

```
source.c:8:10: warning: implicit declaration of function 'printf'
```

You forgot to `#include <stdio.h>`.

**Linker errors:**

```
undefined reference to `my_function`
```

The function is declared (in a header) but not defined anywhere.

---

## Complete Example: Seeing Each Stage

**source.c:**

```c
#include <stdio.h>

int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

**Preprocess (gcc -E):**

```c
# 1 "source.c"
# 1 "<built-in>"
# 1 "<command-line>"
# 1 "source.c"
# 1 "/usr/include/stdio.h" 1 3 4
# ... (thousands of lines of stdio.h expansion)
int main(void) {
    printf("Hello, World!\n");
    return 0;
}
```

**Assembly (gcc -S):**

```assembly
main:
    pushq   %rbp
    movq    %rsp, %rbp
    leaq    .LC0(%rip), %rdi
    call    puts@PLT
    movl    $0, %eax
    popq    %rbp
    ret
```

**Object (objdump -d):**

```
0000000000000000 <main>:
   0:   55                      push   %rbp
   1:   48 89 e5                mov    %rsp,%rbp
   4:   48 8d 3d 00 00 00 00    lea    0x0(%rip),%rdi
   b:   e8 00 00 00 00          call   10 <main+0x10>
  10:   b8 00 00 00 00          mov    $0x0,%eax
  15:   5d                      pop    %rbp
  16:   c3                      ret
```

The addresses `0x0` in the `call` instruction are placeholders; the linker fills them with the actual address of `printf` when creating the executable.

---

## Cross-References

- **Previous:** [02 — Program Structure](02-program-structure.md)
- **Next:** [04 — Basic Types](04-basic-types.md)
- **Preprocessor details:** [Part 08 — Preprocessor](/08-preprocessor/notes/52-preprocessor.md)
- **Linker details:** [Part 10 — Libraries and Linking](/10-libraries/notes/64-static-libraries.md)
- **Debugging with object files:** [77 — GDB](/12-tools/notes/77-gdb.md)

---

## References

- ISO/IEC 9899:2018 §5.1.1 — Translation phases
- `man gcc` — GNU Compiler Collection documentation
- `man ld` — GNU Linker documentation
- Levine, J. R. (1999). *Linkers and Loaders*