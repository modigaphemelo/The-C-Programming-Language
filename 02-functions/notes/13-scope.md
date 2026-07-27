# 13: Scope and Lifetime — Local, Global, Static

---

## Overview

Scope determines where a variable is visible. Lifetime determines how long it exists. These are separate concepts. A variable can be visible in one part of the program while existing in another. Understanding both is essential for writing correct C.

---

## Scope

Scope is the region of the program where a variable can be accessed by name.

### Block Scope

Variables declared inside a block `{ }` are visible only within that block.

```c
{
    int x = 10;    // x is visible only inside this block
}
// x is not visible here
```

This applies to function bodies, if statements, loops, and any other block.

**Nested blocks can shadow outer variables:**

```c
int x = 10;
{
    int x = 20;    // Shadowing: this x hides the outer one
    printf("%d", x);    // 20
}
printf("%d", x);        // 10
```

### Function Scope

Function parameters and local variables are visible throughout the function body.

```c
void func(int param) {
    int local = 10;
    // param and local are visible here
}
// param and local are not visible here
```

### File Scope

Variables declared outside any function have file scope. They are visible from the point of declaration to the end of the file.

```c
int global = 10;        // File scope, visible throughout this file

void func1(void) {
    global = 20;        // Valid
}

void func2(void) {
    global = 30;        // Valid
}
```

### Function Prototype Scope

Parameters declared in function prototypes are only visible within the prototype:

```c
void func(int x, int y);    // x and y are only visible here
```

### Linkage

Linkage determines whether a variable can be accessed from other files.

| Linkage | Keyword | Scope | Visibility Across Files |
|---|---|---|---|
| External | (none) or `extern` | File scope | Yes (unless static) |
| Internal | `static` (file scope) | File scope | No |
| None | (local variables) | Block scope | No |

**External linkage** (default for file-scope variables):

```c
// file1.c
int global = 10;        // External linkage

// file2.c
extern int global;      // Declaration, refers to file1.c's global
```

**Internal linkage** (`static` at file scope):

```c
// file1.c
static int private = 10;    // Internal linkage, not visible outside file1.c

// file2.c
extern int private;         // Error: private is not visible
```

---

## Lifetime (Storage Duration)

Lifetime is the period during which a variable exists in memory.

### Automatic Duration

- Local variables (non-static)
- Created when the block is entered
- Destroyed when the block is exited
- No initialization by default (garbage values)

```c
void func(void) {
    int x;              // Created when func is called
    int y = 5;          // Created and initialized
    // x and y are destroyed when func returns
}
```

### Static Duration

- Variables declared with `static` or at file scope
- Created at program startup
- Destroyed at program exit
- Initialized to zero if no explicit initializer

```c
static int s = 10;          // Static duration, visible only in this file
int global = 20;            // Static duration, visible across files
```

```c
void counter(void) {
    static int count = 0;   // Initialized once, retains value
    count++;
    return count;
}
```

### Thread Duration (C11)

- Variables declared with `_Thread_local`
- Created when the thread starts
- Destroyed when the thread exits

```c
_Thread_local int thread_var = 0;    // One copy per thread
```

### Dynamic Duration

- Allocated with `malloc()`, `calloc()`, or `realloc()`
- Created when allocated
- Destroyed when `free()` is called
- Programmer controls the lifetime

```c
int *p = malloc(sizeof(int));    // Dynamic duration
*p = 42;
free(p);                         // Destroyed
```

---

## Summary Table

| Storage Class | Scope | Lifetime | Initial Value | Linkage |
|---|---|---|---|---|
| `auto` (default for locals) | Block | Automatic | Garbage | None |
| `static` (local) | Block | Static | Zero | None |
| `static` (file scope) | File | Static | Zero | Internal |
| `extern` (declaration) | File | Static | N/A | External |
| `register` | Block | Automatic | Garbage | None |
| `_Thread_local` (C11) | Block/File | Thread | Zero | Internal/External |

---

## Common Pitfalls

### 1. Returning Pointer to Local Variable

```c
int *get_value(void) {
    int x = 42;
    return &x;      // Dangerous: x is destroyed
}
```

### 2. Using Global Variables for No Reason

```c
int temp;           // Global

void swap(int *a, int *b) {
    temp = *a;      // Unnecessary global
    *a = *b;
    *b = temp;
}
```

**Fix:** Use local variables.

### 3. Forgetting `extern` for Global Variables in Headers

```c
// header.h
int global = 10;    // This is a definition, not a declaration
```

**Fix:**

```c
// header.h
extern int global;    // Declaration

// file1.c
int global = 10;      // Definition
```

### 4. Shadowing Variables Unintentionally

```c
int x = 10;

void func(void) {
    int x = 20;     // Shadows global x
    printf("%d", x);    // 20
}
```

---

## Complete Example

```c
#include <stdio.h>

// File scope (static duration, external linkage)
int global = 10;

// File scope (static duration, internal linkage)
static int file_static = 20;

// Static function (internal linkage)
static void print_message(const char *msg) {
    printf("%s\n", msg);
}

// Function with static local variable
int counter(void) {
    static int count = 0;    // Static duration, block scope
    count++;
    return count;
}

void demonstrate_local(void) {
    int local = 5;           // Automatic duration, block scope
    printf("local: %d\n", local);
}

void demonstrate_shadowing(void) {
    int global = 100;        // Shadows the file-scope global
    printf("Inside shadowing: global = %d\n", global);
}

int main(void) {
    printf("global: %d\n", global);
    printf("file_static: %d\n", file_static);
    
    print_message("Hello from static function");
    
    printf("counter: %d\n", counter());    // 1
    printf("counter: %d\n", counter());    // 2
    printf("counter: %d\n", counter());    // 3
    
    demonstrate_local();
    
    demonstrate_shadowing();
    printf("Outside shadowing: global = %d\n", global);    // Still 10
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [12 — Function Prototypes](12-prototypes.md)
- **Next:** [14 — Recursion](14-recursion.md)
- **Dynamic allocation:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)
- **Thread-local storage:** [75 — Threads](/11-system/notes/75-threads.md)

---

## References

- ISO/IEC 9899:2018 §6.2.2 — Linkages of identifiers
- ISO/IEC 9899:2018 §6.2.4 — Storage durations of objects
- ISO/IEC 9899:2018 §6.7.1 — Storage-class specifiers