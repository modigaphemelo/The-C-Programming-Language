
# 10: Function Syntax — Return Type, Parameters, Body

---

## Overview

Functions are the fundamental unit of organization in C. They encapsulate behavior, enable reuse, and provide the building blocks for abstraction. Without functions, your code is a linear script. With functions, it becomes a system.

A function consists of:

- **Return type** — what the function sends back (or `void` if nothing)
- **Name** — how you call it
- **Parameters** — what it receives (or `void` if nothing)
- **Body** — the code that executes

---

## Function Anatomy

```c
return_type function_name(parameter_list) {
    // Body
    // ...
    return value;  // if return_type is not void
}
```

**Example:**

```c
int add(int a, int b) {
    return a + b;
}
```

---

## Return Types

The return type can be any C type: `int`, `char`, `float`, `double`, pointers, structs, unions, or `void`.

### `void` Functions

A function that returns nothing uses `void`:

```c
void print_hello(void) {
    printf("Hello\n");
}
```

You can use `return;` to exit early:

```c
void log_message(int level, const char *msg) {
    if (level < 0) {
        return;          // Exit early for invalid level
    }
    printf("[%d] %s\n", level, msg);
}
```

### Returning Values

```c
int square(int x) {
    return x * x;
}

double average(double a, double b) {
    return (a + b) / 2.0;
}
```

The return expression must match the return type. Implicit conversions are allowed but may lose information:

```c
int divide(int a, int b) {
    return a / b;        // Integer division, truncates
}
```

---

## Parameters

Parameters are variables that receive values when the function is called.

```c
int add(int a, int b) {
    return a + b;
}
```

When you call `add(5, 3)`, `a` becomes `5` and `b` becomes `3`. The values are **copied**—the function receives its own copy. This is pass-by-value.

### `void` Parameters

```c
int get_random_number(void) {
    return 4;  // chosen by fair dice roll
}
```

`void` explicitly states: "This function takes no arguments." In C, `int get_random_number()` means "unknown number of arguments"—a dangerous legacy behavior.

**Always use `(void)` for functions with no parameters.**

### Function Parameters vs Local Variables

Parameters behave like local variables:

```c
int func(int x) {
    int y = 10;
    return x + y;    // x from parameter, y from local
}
```

---

## Pass by Value

C passes arguments by value. The function receives a copy of the original value.

```c
void increment(int x) {
    x++;      // Only modifies the local copy
}

int main(void) {
    int a = 5;
    increment(a);      // a remains 5
    return 0;
}
```

To modify the original value, pass a pointer:

```c
void increment(int *x) {
    (*x)++;   // Modifies the original value
}

int main(void) {
    int a = 5;
    increment(&a);     // a becomes 6
    return 0;
}
```

---

## Function Prototypes

A prototype is a declaration that tells the compiler about the function before it is defined.

```c
int add(int a, int b);    // Prototype

int main(void) {
    int sum = add(5, 3);  // Works: compiler knows add exists
    return 0;
}

int add(int a, int b) {   // Definition
    return a + b;
}
```

**Without a prototype**, the compiler assumes the function returns `int` and takes unknown arguments. This is legacy behavior and should be avoided.

**Headers** are the standard way to provide prototypes:

```c
// math.h
int add(int a, int b);
int subtract(int a, int b);
```

---

## Function Definitions

A definition provides the full implementation. It includes the body.

```c
// Declaration (prototype) — in header
int add(int a, int b);

// Definition (implementation) — in .c file
int add(int a, int b) {
    return a + b;
}
```

**Multiple definitions** are not allowed. Define a function once.

---

## Scope and Lifetime

- **Parameters and local variables** — block scope, automatic duration
- **Static local variables** — block scope, static duration
- **Global variables** — file scope, static duration

See [05 — Variables](05-variables.md) for details.

---

## Recursion

A function can call itself:

```c
int factorial(int n) {
    if (n <= 1) {
        return 1;           // Base case
    }
    return n * factorial(n - 1);   // Recursive case
}
```

**Requirements:**

- Base case (stops the recursion)
- Progress toward the base case

**Cost:** Each recursive call consumes stack space. Deep recursion can cause stack overflow. For production code, prefer iteration when possible.

---

## Inline Functions (C99)

`inline` suggests to the compiler that the function should be expanded in place, avoiding the overhead of a function call:

```c
static inline int max(int a, int b) {
    return (a > b) ? a : b;
}
```

The compiler may ignore the hint. Inline functions are best for small, frequently called functions.

---

## Common Pitfalls

### 1. Missing Return

```c
int add(int a, int b) {
    // No return statement — undefined behavior
}
```

**Fix:** Always return a value for non-`void` functions.

### 2. Function Call with Wrong Types

```c
int add(int a, int b);
double result = add(5.5, 3.2);   // 5.5 and 3.2 are truncated to int
```

### 3. Returning a Pointer to a Local Variable

```c
int *get_value(void) {
    int x = 5;
    return &x;          // Dangerous: x is destroyed when function returns
}
```

**Fix:** Use `static`, dynamic allocation, or pass a pointer to the caller's variable.

### 4. Forgetting Prototypes

Without prototypes, the compiler assumes `int` return type and unknown arguments:

```c
double square(double x);   // Prototype
double result = square(5.0);   // Correct

double square(double x) {   // Definition
    return x * x;
}
```

---

## Complete Example

```c
#include <stdio.h>

// Prototypes
int add(int a, int b);
int factorial(int n);
void print_array(int arr[], int size);
void swap(int *a, int *b);

int main(void) {
    int x = 5, y = 3;
    
    printf("%d + %d = %d\n", x, y, add(x, y));
    printf("%d! = %d\n", 5, factorial(5));
    
    int arr[] = {1, 2, 3, 4, 5};
    print_array(arr, 5);
    
    printf("Before swap: x = %d, y = %d\n", x, y);
    swap(&x, &y);
    printf("After swap: x = %d, y = %d\n", x, y);
    
    return 0;
}

// Definitions
int add(int a, int b) {
    return a + b;
}

int factorial(int n) {
    if (n <= 1) {
        return 1;
    }
    return n * factorial(n - 1);
}

void print_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}
```

---

## Project: Function Library

Create a simple function library:

1. `math_utils.c/h` — functions for common operations
2. `string_utils.c/h` — string utilities
3. `main.c` — test the functions

---

## Cross-References

- **Previous:** [09 — Loops](09-loops.md)
- **Next:** [11 — Pass by Value](11-pass-by-value.md)
- **Pointers and functions:** [27 — Pointer Parameters](04-pointers/notes/27-pointer-parameters.md)
- **Static functions:** [13 — Scope and Lifetime](02-functions/notes/13-scope.md)
- **Inline functions:** [15 — Inline Functions](02-functions/notes/15-inline.md)
- **Variadic functions:** [16 — Variadic Functions](02-functions/notes/16-variadic.md)

---

## References

- ISO/IEC 9899:2018 §6.7.6.3 — Function declarators
- ISO/IEC 9899:2018 §6.9.1 — Function definitions