# 02: Structure of a C Program — `main()`, Headers

**Context:** Every C program follows a structural template defined by the compiler and linker. This is not merely ceremonial—the structure directly reflects how the operating system loads and executes the program, how the linker resolves symbols, and how the preprocessor handles declarations.

---

## The Minimal Program

    int main(void) {
        return 0;
    }

This compiles and runs. Every character has meaning:

- `int` — The program returns an integer to the operating system. Convention: `0` means success, non-zero indicates failure.
- `main` — The entry point. The linker looks for this symbol. Without it, the linker errors: `undefined reference to 'main'`.
- `(void)` — Explicitly says: takes no arguments. In C, an empty `()` means "any number of arguments of unknown type"—a historical artifact. Modern C uses `(void)` for "truly no parameters."
- `return 0;` — Exits the program, passing `0` to the OS. Execution reaches the closing brace? In C99+, `main` implicitly returns `0`. Explicit is clearer.

---

## Program Anatomy

A complete C program consists of:

    // 1. Documentation/comments (optional but professional)
    /*
     * File:   example.c
     * Purpose: Demonstrates program structure
     */
    
    // 2. Preprocessor directives
    #include <stdio.h>
    #include "myheader.h"
    #define BUFFER_SIZE 256
    
    // 3. Global declarations
    extern int errno;
    static const double PI = 3.14159;
    
    // 4. Function prototypes (declarations)
    void process_data(int input);
    int validate_input(double value);
    
    // 5. Main function
    int main(int argc, char *argv[]) {
        // Implementation
        process_data(42);
        return 0;
    }
    
    // 6. Function definitions
    void process_data(int input) {
        // Implementation
    }
    
    int validate_input(double value) {
        // Implementation
    }

The order is not arbitrary. The compiler processes the file top-to-bottom; identifiers must be declared before use.

---

## The `main` Function Signatures

The standard allows two forms:

    int main(void);                          // No command-line arguments
    int main(int argc, char *argv[]);        // With arguments
    // Or equivalently:
    int main(int argc, char **argv);

Some implementations (non-standard) allow a third with environment variables:

    int main(int argc, char *argv[], char *envp[]);

**Parameters:**

- `argc` — Argument count (minimum 1; the program name is `argv[0]`)
- `argv` — Argument vector. Null-terminated array of strings.
- `envp` — Environment variables. Null-terminated array of `"KEY=value"` strings.

The OS constructs `argv` and `envp` before calling `main`. The stack contains these pointers; `main` simply receives them.

---

## Headers: Why They Exist

Headers are not modules. They are textual inclusion.

    #include <stdio.h>

The preprocessor literally copies the contents of `stdio.h` into your file at that point. This is why:

- **Declarations must be visible.** To call `printf`, the compiler needs to know it exists and what arguments it takes. The header provides the *declaration* without the *definition* (which is in the C standard library, linked later).
- **Sharing between files.** If two `.c` files need the same constants, structs, or prototypes, put them in a header.
- **Encapsulation.** Headers expose the public interface; implementations hide in `.c` files.

**Angled brackets vs quotes:**

- `#include <stdio.h>` — Search system include paths first.
- `#include "myheader.h"` — Search current directory first, then system paths.

---

## Translation Unit

The fundamental concept: a **translation unit** is a source file after preprocessing (all `#include`s expanded, all macros substituted, conditional compilation resolved). The compiler compiles one translation unit at a time, producing an object file (`.o` or `.obj`).

The linker later combines object files, resolving references between them.

This explains:

- Why you need headers: each translation unit must see declarations of functions defined elsewhere.
- Why static functions are private: they are not visible to the linker across translation units.
- Why global variables with the same name in different files conflict: the linker sees duplicate symbols.

---

## Program Startup and Termination

**Before `main`:**

1. OS loads the executable into memory.
2. The runtime startup code (`crt0.o` or similar) runs:
   - Initializes static data
   - Sets up `argc`/`argv` on the stack
   - Calls `main`

**After `main` returns:**

1. `main`'s return value is passed to `exit()` (implicitly).
2. `exit()`:
   - Calls functions registered with `atexit()`
   - Flushes I/O buffers
   - Closes open files
   - Returns exit status to the OS

You can also call `exit()` or `_Exit()` explicitly anywhere in the program.

---

## Complete Example With All Elements

    /**
     * program.c - Demonstrates full program structure
     */
    
    #include <stdio.h>
    #include <stdlib.h>
    
    #define MAX_INPUT 1024
    
    static int internal_counter = 0;
    
    void process_line(const char *line);
    void print_summary(void);
    
    int main(int argc, char *argv[]) {
        FILE *input = stdin;
        
        if (argc > 1) {
            input = fopen(argv[1], "r");
            if (!input) {
                perror("fopen");
                return 1;
            }
        }
        
        char buffer[MAX_INPUT];
        while (fgets(buffer, sizeof(buffer), input)) {
            process_line(buffer);
        }
        
        if (input != stdin) fclose(input);
        
        print_summary();
        return 0;
    }
    
    void process_line(const char *line) {
        internal_counter++;
        printf("%4d: %s", internal_counter, line);
    }
    
    void print_summary(void) {
        printf("\n---\nTotal lines: %d\n", internal_counter);
    }

---

## Cross-References

- **Previous:** [01 — History](01-history.md)
- **Next:** [03 — Compilation](03-compilation.md)
- **Linking:** See [Part 10: Libraries and Linking](/10-libraries/notes/64-static-libraries.md) for how multiple translation units combine
- **Preprocessor:** See [Part 08: Preprocessor](/08-preprocessor/notes/52-preprocessor.md) for details on `#include` and macros
- **Program startup:** Embedded systems often replace `crt0` with custom startup code (see [98 — Embedded C](/15-real-world/notes/98-embedded.md))

---

## References

- ISO/IEC 9899:2018 (C17) §5.1.2 — Program execution
- ISO/IEC 9899:2018 §5.1.2.2 — Program startup
- `man 3 exit` — Program termination
