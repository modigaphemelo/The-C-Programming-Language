# 15: Inline Functions — Hinting the Compiler (C99)

---

## Overview

An `inline` function is a function where the compiler is *suggested* to insert the function's code directly at the call site, rather than generating a call instruction. This eliminates the overhead of a function call and can improve performance for small, frequently called functions.

**Key points:**

- `inline` is a hint, not a command
- The compiler may ignore it
- Inline functions must be defined in the same translation unit where they are used (usually in headers)
- In C99 and later, `inline` has specific semantics related to linkage

---

## Syntax

```c
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}
```

**Important:** In practice, `static inline` is the most common and portable form.

---

## Why Inline Functions Exist

Function calls have overhead:

- Push arguments onto the stack
- Save registers
- Jump to the function
- Execute the function
- Restore registers
- Return

For small functions, the overhead can be larger than the work the function does.

**Without inline:**

```c
int max(int a, int b) {
    return (a > b) ? a : b;
}

int main(void) {
    int x = max(5, 3);    // Function call overhead
    return 0;
}
```

**With inline (compiler may generate):**

```c
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}

int main(void) {
    int x = (5 > 3) ? 5 : 3;    // Inlined: no function call
    return 0;
}
```

---

## The `inline` Keyword in C99

C99 introduced `inline` with specific semantics. There are three forms:

### 1. `static inline`

```c
static inline int add(int a, int b) {
    return a + b;
}
```

- Each translation unit gets its own copy
- No external linkage
- Safe to put in headers
- Most portable and widely used

### 2. `inline` without `static`

```c
inline int add(int a, int b) {
    return a + b;
}
```

- External linkage
- Exactly one translation unit must provide an external definition
- Complex and error-prone

### 3. `extern inline`

```c
extern inline int add(int a, int b) {
    return a + b;
}
```

- Forces an external definition
- Rarely used

**Recommendation:** Use `static inline`. It works everywhere and avoids linker issues.

---

## Where to Place Inline Functions

Because the compiler needs to see the function body to inline it, inline functions are typically defined in headers:

```c
// math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

static inline int max(int a, int b) {
    return (a > b) ? a : b;
}

static inline int min(int a, int b) {
    return (a < b) ? a : b;
}

static inline int abs_int(int x) {
    return (x < 0) ? -x : x;
}

#endif
```

```c
// main.c
#include "math_utils.h"

int main(void) {
    int x = max(10, 20);
    int y = abs_int(-5);
    return 0;
}
```

---

## When to Use Inline Functions

| Situation | Recommendation |
|---|---|
| Very small functions (1-5 lines) | Good candidate |
| Called many times in loops | Good candidate |
| Performance-critical code | Good candidate |
| Large functions | Avoid (compiler will ignore anyway) |
| Called from many files | Use `static inline` in header |
| Embedded systems with speed constraints | Consider carefully |

---

## When the Compiler Ignores `inline`

The compiler may ignore `inline` if:

- The function is too large
- The function is recursive
- The function is variadic
- The function's address is taken
- The compiler simply decides not to inline

---

## Inline vs Macros

Before inline functions, macros were the only way to avoid function call overhead.

| Aspect | Inline Function | Macro |
|---|---|---|
| Type safety | Yes | No (text substitution) |
| Scope | Yes | No (macro scope) |
| Debugging | Yes (symbols visible) | No |
| Side effects | Safe | Unsafe (evaluated multiple times) |
| Syntax | Normal function syntax | Preprocessor syntax |
| Code size | Can increase code size | Can increase code size |

```c
// Macro (dangerous)
#define MAX(a, b) ((a) > (b) ? (a) : (b))

// Inline function (safe)
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}
```

The inline function wins on all counts. Use it instead of function-like macros when possible.

---

## Common Pitfalls

### 1. Using `inline` in a `.c` File Without `static`

```c
// math.c
inline int add(int a, int b) {    // May cause linker errors
    return a + b;
}
```

**Fix:** Use `static inline` or provide an external definition.

### 2. Expecting the Compiler to Always Inline

```c
static inline int large_function(int x) {
    // 50 lines of code
}
```

The compiler will likely ignore the hint.

### 3. Using Inline Functions Recursively

```c
static inline int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);    // Compiler will likely ignore inline
}
```

### 4. Taking the Address of an Inline Function

```c
static inline int add(int a, int b) {
    return a + b;
}

int (*func_ptr)(int, int) = add;    // Prevents inlining
```

---

## Compiler-Specific Inline Hints

| Compiler | Attribute |
|---|---|
| GCC/Clang | `__attribute__((always_inline))` |
| MSVC | `__forceinline` |

These force the compiler to inline (if possible):

```c
// GCC/Clang
static inline int add(int a, int b) __attribute__((always_inline)) {
    return a + b;
}

// MSVC
__forceinline int add(int a, int b) {
    return a + b;
}
```

**Use sparingly.** The compiler usually knows better than you do.

---

## Complete Example

```c
#include <stdio.h>

// Simple inline function
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}

static inline int square(int x) {
    return x * x;
}

static inline int clamp(int value, int min, int max) {
    if (value < min) return min;
    if (value > max) return max;
    return value;
}

// This will likely not be inlined
static inline int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

int main(void) {
    int a = 5, b = 10;
    int c = max(a, b);          // Likely inlined
    int d = square(5);          // Likely inlined
    int e = clamp(100, 0, 50);  // Likely inlined
    int f = factorial(5);       // Probably not inlined
    
    printf("max: %d\n", c);
    printf("square: %d\n", d);
    printf("clamp: %d\n", e);
    printf("factorial: %d\n", f);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [14 — Recursion](14-recursion.md)
- **Next:** [16 — Variadic Functions](16-variadic.md)
- **Macros:** [53 — Macros](/08-preprocessor/notes/53-macros.md)
- **Function pointers:** [29 — Function Pointers](/04-pointers/notes/29-function-pointers.md)

---

## References

- ISO/IEC 9899:2018 §6.7.4 — Function specifiers
- GCC: `__attribute__((always_inline))`
- MSVC: `__forceinline`