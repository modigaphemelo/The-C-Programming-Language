# 32: Static Allocation — Compile-time Fixed Size

---

## Overview

Static allocation is memory that is allocated at compile time and exists for the entire duration of the program. It is the simplest form of memory allocation—the size is fixed, and the lifetime is the entire program execution.

**Key characteristics:**

- Allocated at compile time
- Exists for the entire program lifetime
- Size is fixed (cannot change)
- Zero-initialized by default (if not explicitly initialized)
- No runtime overhead
- Cannot be freed

---

## Types of Static Allocation

### 1. Global Variables

Variables declared outside any function have static storage duration.

```c
int global = 10;          // Static allocation
static int file_private;  // Static allocation (file scope)
```

### 2. Static Local Variables

Variables declared with `static` inside a function have static storage duration.

```c
void counter(void) {
    static int count = 0;    // Static allocation
    count++;
    return count;
}
```

### 3. String Literals

String literals are stored in static memory (usually read-only).

```c
char *s = "Hello";    // "Hello" is statically allocated
```

---

## Initialization

Static variables are initialized **once** at program startup.

**Rule:** If you don't explicitly initialize a static variable, it is zero-initialized.

```c
static int x;        // x = 0
static int y = 10;   // y = 10
static int arr[5];   // All zeros
```

**Important:** Static local variables are initialized only once, even if the function is called multiple times.

```c
void func(void) {
    static int count = 0;    // Initialized once
    count++;
    printf("%d\n", count);   // 1, 2, 3, ...
}
```

---

## Storage Duration Comparison

| Aspect | Static | Stack | Heap |
|---|---|---|---|
| Lifetime | Entire program | Function scope | Until freed |
| Allocation time | Compile time | Runtime (function entry) | Runtime (malloc) |
| Size | Fixed | Fixed | Variable |
| Initialization | Zero by default | Garbage | Uninitialized |
| Speed | Fast | Fast | Slower |
| Can be freed | No | Automatic | Manual |

---

## When to Use Static Allocation

| Situation | Recommendation |
|---|---|
| Constants that never change | Static (global `const`) |
| Large arrays of fixed size | Static (if lifetime matches program) |
| Shared data across functions | Static (global or file scope) |
| Counters that persist | Static local |
| Data that must be preserved between calls | Static local |
| Configuration data | Static (read-only) |

---

## Static vs Global

```c
// Global variable (visible to other files)
int global = 10;

// Static global variable (visible only in this file)
static int file_private = 20;

// Static local variable (visible only in function)
void func(void) {
    static int local_static = 30;
}
```

**Linkage:**

| Declaration | Scope | Linkage |
|---|---|---|
| `int global;` | File | External |
| `static int file_private;` | File | Internal |
| `static int local_static;` | Block | None |

---

## Static Variables in Functions

Static local variables retain their value between function calls. This is useful for:

- Counters
- State machines
- Caching
- Singleton-like patterns

```c
int get_next_id(void) {
    static int id = 0;    // Persists between calls
    return id++;
}

int main(void) {
    printf("%d\n", get_next_id());    // 0
    printf("%d\n", get_next_id());    // 1
    printf("%d\n", get_next_id());    // 2
    return 0;
}
```

**Caution:** Static local variables are not thread-safe. Each thread would share the same variable.

---

## Static Functions

Functions declared with `static` are visible only in the file where they are defined.

```c
// helper.c
static int private_function(int x) {
    return x * 2;
}

int public_function(int x) {
    return private_function(x) + 1;
}
```

`private_function` is not visible outside `helper.c`.

---

## Static Arrays

Static arrays are allocated at compile time and exist for the entire program.

```c
static int buffer[1024];    // Fixed size, entire program lifetime
```

**Advantages:**
- No allocation overhead
- No risk of memory leaks
- Fast access

**Disadvantages:**
- Fixed size (cannot grow)
- Memory is allocated even if not used
- May waste memory

---

## Common Pitfalls

### 1. Confusing Static Duration with Stack Duration

```c
int *get_value(void) {
    static int x = 42;    // OK: returns valid pointer
    return &x;
}

int *get_value_bad(void) {
    int x = 42;           // BAD: stack variable
    return &x;            // Dangling pointer
}
```

### 2. Returning a Pointer to a Static Variable

```c
char *get_string(void) {
    static char buffer[100];
    strcpy(buffer, "Hello");
    return buffer;    // OK: buffer is static
}
```

**But be careful:** The same buffer is shared across all calls.

```c
char *s1 = get_string();    // "Hello"
char *s2 = get_string();    // "World"
// s1 now points to "World" too (shared buffer)
```

### 3. Forgetting That Static Variables Are Zero-Initialized

```c
static int x;    // x = 0 (not garbage)
```

### 4. Using Static for Large Arrays on the Stack (Wrong)

```c
void func(void) {
    static int arr[1000000];    // OK: static, not stack
    int arr2[1000000];          // BAD: stack overflow
}
```

### 5. Thread Safety Issues

```c
void counter(void) {
    static int count = 0;    // Shared across threads
    count++;                 // Not thread-safe
}
```

---

## Complete Example

```c
#include <stdio.h>
#include <string.h>

// Global (static)
int global_counter = 0;

// File-private static
static int file_counter = 0;

// Static local variable
int get_next_id(void) {
    static int id = 0;
    return id++;
}

// Static function (file-private)
static void increment_file_counter(void) {
    file_counter++;
}

// Returning pointer to static buffer
char *get_message(void) {
    static char buffer[100];
    strcpy(buffer, "Hello from static buffer");
    return buffer;
}

// Counting with static local
void count_calls(const char *name) {
    static int calls = 0;    // Persists between calls
    calls++;
    printf("%s called %d times\n", name, calls);
}

int main(void) {
    // Global
    global_counter++;
    printf("global_counter: %d\n", global_counter);
    
    // File-static
    increment_file_counter();
    increment_file_counter();
    // file_counter is 2 (but not directly accessible here)
    
    // Static local
    printf("id: %d\n", get_next_id());    // 0
    printf("id: %d\n", get_next_id());    // 1
    printf("id: %d\n", get_next_id());    // 2
    
    // Static buffer
    char *msg = get_message();
    printf("msg: %s\n", msg);
    
    // Static buffer is shared!
    char *msg2 = get_message();    // Returns same buffer
    strcpy(msg2, "Modified");
    printf("msg: %s\n", msg);      // "Modified" (same buffer)
    
    // Counting calls
    count_calls("func1");     // 1
    count_calls("func1");     // 2
    count_calls("func2");     // 3 (shared counter)
    
    // Static vs stack
    static int static_arr[1000];    // BSS segment
    int stack_arr[1000];            // Stack
    
    printf("\nstatic_arr (BSS): %p\n", (void *)static_arr);
    printf("stack_arr (stack): %p\n", (void *)stack_arr);
    
    return 0;
}
```

---

## Summary: Static Allocation Rules

| Declaration | Storage Duration | Scope | Linkage | Initial Value |
|---|---|---|---|---|
| `int x;` (global) | Static | File | External | 0 |
| `static int x;` (global) | Static | File | Internal | 0 |
| `static int x;` (local) | Static | Block | None | 0 |
| `int x;` (local) | Automatic | Block | None | Garbage |
| `const char *s = "Hello";` | Static | File | External | "Hello" |

---

## Cross-References

- **Previous:** [31 — Stack vs Heap](31-stack-heap.md)
- **Next:** [33 — Dynamic Allocation](33-dynamic-allocation.md)
- **Global variables:** [05 — Variables](05-variables.md)
- **Scope and lifetime:** [13 — Scope and Lifetime](13-scope.md)
- **Memory layout:** [36 — Memory Layout](36-memory-layout.md)

---

## References

- ISO/IEC 9899:2018 §6.2.4 — Storage durations of objects
- ISO/IEC 9899:2018 §6.7.1 — Storage-class specifiers
- ISO/IEC 9899:2018 §6.7.9 — Initialization