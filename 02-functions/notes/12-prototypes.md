# 12: Function Prototypes — Declaration vs Definition

---

## Overview

A function prototype tells the compiler about a function before it is used. It declares the function's name, return type, and parameter types. The actual implementation (the definition) can appear later—in the same file, in another file, or in a library.

Without a prototype, the compiler makes assumptions that are often wrong. This leads to subtle bugs. Prototypes are not optional in modern C. They are mandatory for correct code.

---

## Declaration vs Definition

The distinction is critical:

| Concept | What It Does | Memory Allocated? |
|---|---|---|
| Declaration | Introduces the function's name and type | No |
| Definition | Provides the implementation | Yes (code) |

**Declaration (prototype):**

```c
int add(int a, int b);
```

**Definition:**

```c
int add(int a, int b) {
    return a + b;
}
```

A declaration can appear multiple times. A definition must appear exactly once.

---

## Why Prototypes Exist

The compiler processes files top to bottom. It needs to know what a function looks like before it can compile a call to it.

```c
int main(void) {
    int result = add(5, 3);    // Compiler error: add not declared
    return 0;
}

int add(int a, int b) {        // Definition appears later
    return a + b;
}
```

**Fix:** Add a prototype before `main`:

```c
int add(int a, int b);          // Prototype

int main(void) {
    int result = add(5, 3);    // Works: compiler knows add exists
    return 0;
}

int add(int a, int b) {
    return a + b;
}
```

---

## Prototype Syntax

```c
return_type function_name(parameter_list);
```

**Parameters can be named or unnamed:**

```c
int add(int a, int b);          // Named parameters (clear)
int add(int, int);              // Unnamed (works but less clear)
```

**Empty parameter list:**

```c
void func();                    // Unknown number of parameters (old style)
void func(void);                // No parameters (correct)
```

**Always use `void` for functions with no parameters.**

---

## Prototypes in Headers

The standard practice is to place prototypes in header files (`.h`):

**math.h:**

```c
#ifndef MATH_H
#define MATH_H

int add(int a, int b);
int subtract(int a, int b);
int multiply(int a, int b);
double divide(double a, double b);

#endif
```

**math.c:**

```c
#include "math.h"

int add(int a, int b) {
    return a + b;
}

int subtract(int a, int b) {
    return a - b;
}

// etc.
```

**main.c:**

```c
#include <stdio.h>
#include "math.h"

int main(void) {
    int sum = add(5, 3);
    printf("%d\n", sum);
    return 0;
}
```

---

## What Happens Without a Prototype

C allows calling a function without a prototype. This is **implicit declaration**. The compiler assumes:

- The function returns `int`
- The arguments are passed according to their types

This can cause serious bugs:

```c
#include <stdio.h>

int main(void) {
    double result = sqrt(2.0);   // sqrt is not declared
    printf("%f\n", result);
    return 0;
}
```

Without `#include <math.h>`, the compiler assumes `sqrt` returns `int`. The actual return value is `double`, but the compiler treats it as `int`. The result is garbage.

**Implicit declarations were removed in C99.** Modern compilers will warn (or error) about them.

---

## Static Functions

Functions declared with `static` are visible only within the file where they are defined:

```c
// helper.c
static int internal_utility(int x) {
    return x * 2;
}

int public_function(int x) {
    return internal_utility(x) + 1;
}
```

`internal_utility` is not visible outside `helper.c`. This is how you create private functions.

---

## Inline Functions (C99)

`inline` suggests to the compiler that the function should be expanded in place:

```c
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}
```

**Best practice:** Use `static inline` in headers for small, frequently called functions. This avoids linker errors when the header is included in multiple files.

---

## Common Pitfalls

### 1. Prototype Mismatch

```c
void func(int x);          // Prototype takes int

void func(double x) {      // Definition takes double
    // Different types: undefined behavior
}
```

### 2. Forgetting Prototypes

```c
int main(void) {
    char *s = "hello";
    int len = strlen(s);   // strlen not declared
    return 0;
}
```

**Fix:** `#include <string.h>`

### 3. Wrong Return Type

```c
double sqrt(double x);     // Correct

int main(void) {
    double result = sqrt(2.0);   // Works
}
```

### 4. Inconsistent `const` in Prototypes

```c
void print(const char *s);    // Prototype

void print(char *s) {         // Definition
    // Different pointer types: UB
}
```

---

## Prototype vs Definition Summary

| Aspect | Prototype | Definition |
|---|---|---|
| Ends with `;` | Yes | No (uses `{}`) |
| Has function body | No | Yes |
| Can appear multiple times | Yes | No |
| Allocates memory | No | Yes |
| Location | Header (usually) | Source file (usually) |
| Visibility | Exposes interface | Hides implementation |

---

## Complete Example

```c
#include <stdio.h>

// Prototypes
int add(int a, int b);
int multiply(int a, int b);
void print_result(int a, int b, int result);

// Static function (private to this file)
static int utility(int x) {
    return x * x;
}

int main(void) {
    int a = 5, b = 3;
    
    int sum = add(a, b);
    print_result(a, b, sum);
    
    int product = multiply(a, b);
    print_result(a, b, product);
    
    // static function is visible here
    printf("utility(4) = %d\n", utility(4));
    
    return 0;
}

// Definitions
int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return a * b;
}

void print_result(int a, int b, int result) {
    printf("%d + %d = %d\n", a, b, result);
}

// static utility defined here
static int utility(int x) {
    return x * x;
}
```

---

## Cross-References

- **Previous:** [11 — Pass by Value](11-pass-by-value.md)
- **Next:** [13 — Scope and Lifetime](13-scope.md)
- **Headers:** [64 — Static Libraries](/10-libraries/notes/64-static-libraries.md)
- **Static functions:** [13 — Scope and Lifetime](13-scope.md)
- **Inline functions:** [15 — Inline Functions](15-inline.md)

---

## References

- ISO/IEC 9899:2018 §6.7.6.3 — Function declarators
- ISO/IEC 9899:2018 §6.9.1 — Function definitions
- ISO/IEC 9899:2018 §6.11 — Implicit function declarations (removed)