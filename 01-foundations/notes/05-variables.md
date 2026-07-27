# 05: Variables — Declaration, Initialization, Scope

---

## Overview

Variables are named storage locations in memory. A variable declaration tells the compiler:

- The name of the variable
- The type of data it holds
- The scope (where it is visible)
- The storage duration (how long it lives)

C gives you precise control over all of these. Understanding variables is understanding how C manages memory.

---

## Declaration

A declaration introduces a variable's name and type. It does not necessarily allocate storage (that happens at definition).

**Syntax:**

```c
type identifier;
type identifier = initial_value;
```

**Examples:**

```c
int x;                  // Declaration without initialization
int y = 10;             // Declaration with initialization
char c = 'A';
float pi = 3.14159f;
double large = 1.0e100;
```

**Multiple declarations:**

```c
int a, b, c;            // Three ints
int a = 1, b = 2, c;    // Mix of initialized and uninitialized
```

---

## Definition

A definition is a declaration that allocates storage. In most cases, declarations are also definitions:

```c
int x = 5;      // Definition (storage allocated)
```

A declaration that is *not* a definition:

```c
extern int x;   // Declaration only — x is defined elsewhere
```

The `extern` keyword tells the compiler that the variable exists in another translation unit. The storage is allocated there, not here.

---

## Initialization

Uninitialized variables contain garbage values (whatever was in that memory location previously). Always initialize variables before use.

**Zero initialization:**

```c
int x = 0;
char c = '\0';
float f = 0.0f;
```

**Global and static variables** are automatically initialized to zero if not explicitly initialized. This is the only case where you can rely on default initialization.

**Automatic variables** (local, non-static) are not initialized. Their initial value is indeterminate and reading them is undefined behavior.

---

## Scope

Scope determines where a variable is visible. C has several types of scope.

### Block Scope

Variables declared inside a block `{ }` are visible only within that block:

```c
{
    int x = 5;          // Visible only inside this block
    // x is valid here
}
// x is not valid here
```

This applies to function bodies, `if` statements, loops, and any other block.

**Nested blocks can shadow variables:**

```c
int x = 10;
{
    int x = 20;         // This x shadows the outer x
    printf("%d", x);    // Prints 20
}
printf("%d", x);        // Prints 10
```

### Function Scope

Variables declared inside a function are visible throughout the function body, but not outside it:

```c
void func(void) {
    int x = 5;          // Visible only inside func()
}
```

### File Scope (Global)

Variables declared outside any function have file scope. They are visible from their declaration point to the end of the file:

```c
int global = 10;        // File scope

void func1(void) {
    printf("%d", global);   // Valid
}

void func2(void) {
    global = 20;            // Valid
}
```

To make a global variable visible across multiple files:

1. Define it once in one `.c` file
2. Declare it with `extern` in the other files (usually via a header)

**file1.c:**

```c
int global = 10;        // Definition
```

**file2.c:**

```c
extern int global;      // Declaration (definition elsewhere)
```

### Function Prototype Scope

Variables declared in function prototypes are only valid within the prototype:

```c
void func(int x, int y);    // x and y are only visible here
```

The parameter names are optional but useful for documentation.

---

## Storage Classes

Storage classes specify the lifetime and scope of variables.

### `auto`

The default for local variables. Rarely used explicitly.

```c
auto int x = 5;         // Same as 'int x = 5;'
```

### `register`

Suggests to the compiler that the variable should be stored in a CPU register for faster access. Modern compilers usually ignore this; they are better at register allocation than humans.

```c
register int counter = 0;
```

You cannot take the address of a `register` variable.

### `static`

**For local variables:** The variable retains its value between function calls. It is initialized once (at program startup) and persists for the entire program lifetime.

```c
int counter(void) {
    static int count = 0;   // Initialized once, retains value
    return ++count;
}
```

**For global variables:** The variable is visible only within the file where it is defined. This is the C equivalent of "private" or "file-static."

```c
static int private = 10;    // Visible only in this file
```

### `extern`

Declares a variable without defining it. Tells the compiler that the variable is defined in another translation unit.

```c
extern int global;          // Declaration only
```

### `_Thread_local` (C11)

Variables that have thread storage duration. Each thread gets its own copy.

```c
_Thread_local int thread_var = 0;
```

---

## Lifetime (Storage Duration)

The lifetime of a variable determines when it is created and destroyed.

### Automatic Duration

- Variables declared inside a block (without `static`)
- Created when the block is entered
- Destroyed when the block is exited
- No guaranteed initialization (garbage values)

```c
void func(void) {
    int x;              // Created when func() is called
    // x is garbage until initialized
    int y = 5;          // Created and initialized
    // y is destroyed when func() returns
}
```

### Static Duration

- Variables declared with `static` or at file scope
- Created at program startup
- Destroyed at program exit
- Initialized to zero if no explicit initializer

```c
static int s = 10;          // Exists for entire program lifetime
int global = 20;            // Same: static duration, file scope
```

### Thread Duration (C11)

- Variables declared with `_Thread_local`
- Created when the thread starts
- Destroyed when the thread exits

### Dynamic Duration

- Allocated with `malloc()`, `calloc()`, or `realloc()`
- Created when allocated
- Destroyed when `free()` is called
- The programmer controls the lifetime

This is covered in detail in [Part 05 — Memory Management](/05-memory/notes/33-dynamic-allocation.md).

---

## Declaration vs Definition: The Difference

This distinction is critical in C.

| Concept | Declares | Defines |
|---|---|---|
| Declaration | Introduces name and type | Does not allocate storage |
| Definition | Introduces name and type | Allocates storage |

```c
// Declaration only
extern int x;
void func(int a);

// Definition
int x = 5;
void func(int a) { /* ... */ }
```

A declaration can appear multiple times. A definition must appear exactly once.

---

## Common Pitfalls

### 1. Using Uninitialized Variables

```c
int x;
printf("%d", x);        // Undefined behavior
```

**Fix:** Always initialize variables.

### 2. Shadowing Variables

```c
int x = 10;
{
    int x = 20;         // Shadows outer x
    // Inside here, x is 20
}
// Outside, x is 10
```

This is legal but confusing. Avoid it.

### 3. Forgetting `static` for File-Private Variables

```c
// file1.c
int private = 5;        // This is visible to other files!

// file2.c
extern int private;     // Can access private from file1.c
```

**Fix:** Use `static` for variables that should not be visible outside the file.

### 4. Misunderstanding `extern`

```c
// header.h
int global;             // This is a definition, not a declaration!

// Two files including header.h will both define global (linker error)
```

**Fix:** Use `extern` in headers and define the variable in one `.c` file.

### 5. Using `register` Incorrectly

```c
register int x;
int *p = &x;            // Error: cannot take address of register variable
```

---

## Complete Example

```c
#include <stdio.h>

// File scope (global)
int global = 10;
static int file_static = 20;

void func_with_static(void) {
    static int count = 0;       // Retains value between calls
    count++;
    printf("count: %d\n", count);
}

void func_with_auto(void) {
    int local = 0;              // New each call, garbage if not initialized
    local++;
    printf("local: %d\n", local);
}

int main(void) {
    // Block scope
    int x = 5;
    printf("x: %d\n", x);
    
    {
        // Inner block
        int x = 10;             // Shadows outer x
        printf("inner x: %d\n", x);
    }
    printf("outer x: %d\n", x); // x is still 5
    
    // Static local variable
    func_with_static();         // count: 1
    func_with_static();         // count: 2
    func_with_static();         // count: 3
    
    // Auto local variable (garbage if uninitialized)
    func_with_auto();           // local: 1
    func_with_auto();           // local: 1
    func_with_auto();           // local: 1
    
    // File scope variables
    printf("global: %d\n", global);
    printf("file_static: %d\n", file_static);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [04 — Basic Types](04-basic-types.md)
- **Next:** [06 — Constants](06-constants.md)
- **Storage duration details:** [31 — Stack vs Heap](/05-memory/notes/31-stack-heap.md)
- **`extern` with multiple files:** [64 — Static Libraries](/10-libraries/notes/64-static-libraries.md)
- **Thread-local storage:** [75 — Threads](/11-system/notes/75-threads.md)

---

## References

- ISO/IEC 9899:2018 §6.2.2 — Linkages of identifiers
- ISO/IEC 9899:2018 §6.2.4 — Storage durations of objects
- ISO/IEC 9899:2018 §6.7.1 — Storage-class specifiers
- ISO/IEC 9899:2018 §6.7.9 — Initialization