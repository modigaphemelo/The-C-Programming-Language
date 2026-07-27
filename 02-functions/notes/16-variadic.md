# 16: Variadic Functions — `stdarg.h`, `printf`-like

---

## Overview

Variadic functions accept a variable number of arguments. The most famous example is `printf`, which can take one format string and any number of additional arguments. C provides a standard way to implement such functions using the `<stdarg.h>` header.

**Use cases:**

- Printf-style formatting functions
- Logging functions
- Error reporting functions
- Functions that operate on a variable number of items

---

## The `<stdarg.h>` Macros

| Macro/Type | Purpose |
|---|---|
| `va_list` | Type for the argument list pointer |
| `va_start` | Initializes the argument list |
| `va_arg` | Retrieves the next argument |
| `va_end` | Cleans up the argument list |
| `va_copy` (C99) | Copies the argument list |

**Basic pattern:**

```c
#include <stdarg.h>

void func(int count, ...) {
    va_list args;
    va_start(args, count);        // Initialize
    for (int i = 0; i < count; i++) {
        int value = va_arg(args, int);    // Get next argument
        // Use value
    }
    va_end(args);                 // Clean up
}
```

---

## The Required Parameter

**A variadic function must have at least one named parameter before the ellipsis (`...`).** This is required by the standard.

```c
void good(int count, ...);       // Correct: named parameter before ...
void bad(...);                   // Error: no named parameter
```

The named parameter is used by `va_start` to know where the variable arguments begin. It is typically the count of arguments or the format string.

---

## `va_start`

```c
va_list args;
va_start(args, last_param);
```

- `args` — the `va_list` variable
- `last_param` — the last named parameter before the ellipsis

`va_start` initializes `args` so that `va_arg` can retrieve the arguments.

---

## `va_arg`

```c
type value = va_arg(args, type);
```

- `args` — the `va_list`
- `type` — the type of the argument to retrieve

`va_arg` returns the next argument of the specified type and advances the internal pointer.

**Important:** The type passed to `va_arg` must match the actual type of the argument. There is no type checking. Passing the wrong type is undefined behavior.

---

## `va_end`

```c
va_end(args);
```

Cleans up the `va_list`. Should be called before the function returns.

---

## `va_copy` (C99)

```c
va_list args2;
va_copy(args2, args);
// Use args2
va_end(args2);
```

Creates a copy of the argument list. Useful when you need to traverse the arguments more than once.

---

## Example: Simple Sum Function

```c
#include <stdarg.h>

int sum(int count, ...) {
    int total = 0;
    va_list args;
    va_start(args, count);
    
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);
    }
    
    va_end(args);
    return total;
}
```

**Usage:**

```c
int result = sum(3, 10, 20, 30);    // result = 60
int result = sum(5, 1, 2, 3, 4, 5); // result = 15
```

---

## Example: Printf-Style Logging

```c
#include <stdarg.h>

void log_message(const char *format, ...) {
    va_list args;
    va_start(args, format);
    vprintf(format, args);    // vprintf uses va_list
    va_end(args);
}

int main(void) {
    log_message("Hello, %s! You are %d years old.\n", "Alice", 30);
    return 0;
}
```

---

## The `vprintf` Family

The standard library provides versions of `printf` and `scanf` that accept a `va_list`:

| Function | Purpose |
|---|---|
| `vprintf(const char *format, va_list ap)` | Print to stdout |
| `vfprintf(FILE *stream, const char *format, va_list ap)` | Print to a file |
| `vsprintf(char *str, const char *format, va_list ap)` | Print to a string |
| `vsnprintf(char *str, size_t size, const char *format, va_list ap)` | Print to a string (with size limit) |

These are useful for building functions that wrap `printf`:

```c
void error(const char *format, ...) {
    va_list args;
    va_start(args, format);
    fprintf(stderr, "ERROR: ");
    vfprintf(stderr, format, args);
    va_end(args);
}
```

---

## Type Safety and `va_arg`

`va_arg` has no type checking. The compiler trusts that you know what you're doing:

```c
int sum(int count, ...) {
    int total = 0;
    va_list args;
    va_start(args, count);
    
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);    // Assumes all arguments are int
    }
    
    va_end(args);
    return total;
}

int main(void) {
    // Wrong: passing double where int is expected
    int result = sum(2, 3.14, 5.67);    // Undefined behavior
    return 0;
}
```

**Solution:** Use a format string (like `printf`) to communicate the types:

```c
void mixed(const char *format, ...) {
    va_list args;
    va_start(args, format);
    
    for (const char *p = format; *p; p++) {
        if (*p == '%') {
            p++;
            switch (*p) {
                case 'd':
                    int i = va_arg(args, int);
                    printf("%d ", i);
                    break;
                case 'f':
                    double d = va_arg(args, double);
                    printf("%f ", d);
                    break;
                case 's':
                    char *s = va_arg(args, char *);
                    printf("%s ", s);
                    break;
            }
        }
    }
    
    va_end(args);
}
```

---

## Common Pitfalls

### 1. No Named Parameter

```c
void bad(...) {    // Error: no named parameter
    // ...
}
```

### 2. Wrong Type in `va_arg`

```c
double value = va_arg(args, double);    // If argument is int, this is UB
```

### 3. Forgetting `va_end`

```c
void func(int count, ...) {
    va_list args;
    va_start(args, count);
    // No va_end
    // Memory may leak on some platforms
}
```

### 4. Passing the Wrong Type to `va_arg` After Using `va_arg`

```c
va_arg(args, int);
va_arg(args, double);    // If the second argument is int, UB
```

### 5. Using `va_arg` After `va_end`

```c
va_end(args);
int value = va_arg(args, int);    // Invalid: args is no longer valid
```

### 6. Passing Arguments of Wrong Type to `printf`-like Functions

```c
printf("%d", 3.14);    // Wrong type: UB
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdarg.h>

// Sum of integers
int sum(int count, ...) {
    int total = 0;
    va_list args;
    va_start(args, count);
    
    for (int i = 0; i < count; i++) {
        total += va_arg(args, int);
    }
    
    va_end(args);
    return total;
}

// Printf-style error logging
void error(const char *format, ...) {
    va_list args;
    va_start(args, format);
    fprintf(stderr, "ERROR: ");
    vfprintf(stderr, format, args);
    va_end(args);
}

// Custom formatting function
void print_formatted(const char *format, ...) {
    va_list args;
    va_start(args, format);
    
    for (const char *p = format; *p; p++) {
        if (*p == '%') {
            p++;
            switch (*p) {
                case 'd': {
                    int val = va_arg(args, int);
                    printf("%d", val);
                    break;
                }
                case 'f': {
                    double val = va_arg(args, double);
                    printf("%f", val);
                    break;
                }
                case 's': {
                    char *val = va_arg(args, char *);
                    printf("%s", val);
                    break;
                }
                case 'c': {
                    char val = (char)va_arg(args, int);
                    printf("%c", val);
                    break;
                }
                default:
                    printf("%%%c", *p);
                    break;
            }
        } else {
            putchar(*p);
        }
    }
    
    va_end(args);
    printf("\n");
}

// Sum with variable types (using format string)
void sum_formatted(const char *format, ...) {
    int total = 0;
    va_list args;
    va_start(args, format);
    
    for (const char *p = format; *p; p++) {
        if (*p == 'd') {
            total += va_arg(args, int);
        }
    }
    
    va_end(args);
    printf("Sum: %d\n", total);
}

int main(void) {
    // Basic sum
    printf("sum(3, 10, 20, 30) = %d\n", sum(3, 10, 20, 30));
    printf("sum(5, 1, 2, 3, 4, 5) = %d\n", sum(5, 1, 2, 3, 4, 5));
    
    // Error logging
    error("File '%s' not found (error %d)\n", "data.txt", 404);
    
    // Custom formatting
    print_formatted("Hello, %s! You have %d messages and %f dollars.\n", 
                    "Alice", 5, 12.34);
    
    // Sum with format string
    sum_formatted("ddd", 10, 20, 30);
    sum_formatted("ddd", 1, 2, 3);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [15 — Inline Functions](15-inline.md)
- **Next:** [Project: Function Library](projects/02-function-library/)
- **`printf`:** [46 — printf](07-io/notes/46-printf.md)
- **`scanf`:** [47 — scanf](07-io/notes/47-scanf.md)
- **`vprintf`:** [45 — stdio](07-io/notes/45-stdio.md)

---

## References

- ISO/IEC 9899:2018 §7.16 — Variable arguments `<stdarg.h>`
- ISO/IEC 9899:2018 §7.21.6.8 — `vprintf`
- ISO/IEC 9899:2018 §7.21.6.9 — `vfprintf`
- ISO/IEC 9899:2018 §7.21.6.10 — `vsprintf`
- ISO/IEC 9899:2018 §7.21.6.11 — `vsnprintf`