# 06: Constants — `const`, `#define`, Enums

---

## Overview

Constants are values that do not change during program execution. C provides multiple ways to define constants, each with different semantics, visibility, and type safety:

| Method | When to Use |
|---|---|
| `#define` | Legacy code, compile-time configuration, simple text replacement |
| `const` | Type-safe constants, pointers to constants, function parameters |
| `enum` | Related integer constants, especially for state machines and flags |
| Literals | Direct values in code |

Understanding the differences is essential. Each method has its place. Using the wrong one creates bugs, reduces readability, or harms performance.

---

## 1. Literal Constants

Literals are values written directly in the source code. They are constants by definition.

**Integer literals:**

```c
42          // int
42U         // unsigned int
42L         // long
42LL        // long long
0x2A        // Hexadecimal (42)
052         // Octal (42) — avoid this syntax
```

**Floating-point literals:**

```c
3.14        // double
3.14f       // float
3.14L       // long double
1e-5        // double (scientific notation)
```

**Character literals:**

```c
'A'         // char (integer value 65)
'\n'        // Newline (ASCII 10)
'\x41'      // Hex escape (65)
'\0'        // Null character (0)
```

**String literals:**

```c
"Hello"     // Array of char, terminated with '\0'
```

String literals are stored in read-only memory. Attempting to modify them is undefined behavior:

```c
char *s = "Hello";
s[0] = 'h';     // Undefined behavior (segmentation fault on most systems)
```

---

## 2. The Preprocessor Method: `#define`

The preprocessor performs textual substitution before compilation.

```c
#define PI 3.14159
#define MAX_BUFFER 1024
#define DEBUG 1
```

**What happens:** Every occurrence of `PI` in the code is replaced with `3.14159` before the compiler sees it.

**Advantages:**

- Works anywhere — function-like macros, conditional compilation, etc.
- No runtime overhead
- Can be used for compile-time configuration

**Disadvantages:**

- No type checking — `PI` is replaced with text, the compiler sees `3.14159`
- No scope — macros are global (until `#undef`)
- Debugging is harder — the debugger sees the expanded value, not the macro name
- Can cause unexpected behavior if not parenthesized correctly

**Common pitfall:**

```c
#define SQUARE(x) x * x

int y = SQUARE(3 + 1);   // Expands to: 3 + 1 * 3 + 1 = 7, not 16
```

**Fix:**

```c
#define SQUARE(x) ((x) * (x))
```

**Always parenthesize macro arguments and the entire expression.**

**Function-like macros:**

```c
#define MAX(a, b) ((a) > (b) ? (a) : (b))
#define MIN(a, b) ((a) < (b) ? (a) : (b))
```

These are useful but dangerous. Each argument is evaluated twice, which can cause side effects:

```c
int x = 5;
int y = MAX(x++, 10);   // x++ evaluated twice — undefined behavior
```

**Predefined macros:**

```c
__FILE__    // Current file name (string literal)
__LINE__    // Current line number (integer)
__DATE__    // Compilation date (string literal)
__TIME__    // Compilation time (string literal)
__func__    // Current function name (string literal) (C99)
__STDC__    // 1 if compiled with a standard C compiler
```

---

## 3. The `const` Keyword

`const` is a type qualifier. It tells the compiler that a variable should not be modified after initialization.

**Basic usage:**

```c
const int MAX = 100;
const float PI = 3.14159f;
```

Unlike `#define`, `const` variables have type and scope. They are checked by the compiler.

**`const` with pointers:**

This is where `const` becomes complex. The position of `const` determines what is constant:

```c
const int *p;        // Pointer to a constant int (the int cannot change)
int const *p;        // Same: pointer to a constant int
int *const p;        // Constant pointer to an int (the pointer cannot change)
const int *const p;  // Constant pointer to a constant int
```

**Rule of thumb:** Read from right to left:

```c
const int *p;        // p is a pointer to an int that is const
int *const p;        // p is a const pointer to an int
```

**`const` in function parameters:**

```c
void print(const char *s) {
    // s points to a constant string — cannot modify *s
    // s itself can be modified (it's a copy)
}
```

This is a promise to the caller: "I will not modify the data you pass to me."

**`const` and casting:**

```c
const int x = 5;
int *p = (int *)&x;     // Dangerous: casting away const
*p = 10;                // Undefined behavior
```

---

## 4. Enumerations (`enum`)

An enumeration creates a set of named integer constants. They are type-safe and scoped.

**Basic `enum`:**

```c
enum Color {
    RED,        // 0
    GREEN,      // 1
    BLUE        // 2
};

enum Color c = RED;
```

**Explicit values:**

```c
enum Status {
    OK = 0,
    ERROR = 1,
    NOT_FOUND = 404,
    TIMEOUT = 100
};
```

**`enum` vs `#define` for constants:**

| Aspect | `enum` | `#define` |
|---|---|---|
| Type safety | Yes (enum type) | No (text substitution) |
| Scope | Block/function scope | File scope |
| Debugger visibility | Yes (symbols are visible) | No (values only) |
| Compile-time evaluation | Yes | Yes |

**Use `enum` for:**

- State machines
- Error codes
- Options/flags
- Related sets of integer constants

**C11: Anonymous enums**

You can omit the tag:

```c
enum {
    STATE_IDLE = 0,
    STATE_RUNNING = 1,
    STATE_STOPPED = 2
};
```

The constants still exist, but the enum type has no name.

---

## 5. Which Method Should You Use?

| Situation | Recommendation |
|---|---|
| Simple numeric constant | `#define` or `const` — both work. Prefer `const` for type safety. |
| String literal | `const char *` |
| Related integer constants | `enum` |
| Compile-time configuration | `#define` (used with `#ifdef`) |
| Function parameters | `const` (gives caller confidence) |
| Pointer that shouldn't change address | `*const` |
| Data that shouldn't be modified | `const *` |

**General rule of thumb:**

- Use `const` for type-safe constants with scope
- Use `enum` for sets of related integer constants
- Use `#define` for compile-time configuration and conditional compilation
- Use `#define` for function-like macros when absolutely necessary (but prefer inline functions in C99+)

---

## Common Pitfalls

### 1. Modifying `const` Variables

```c
const int x = 5;
x = 10;                 // Compiler error
```

### 2. Using `const` with Pointers Incorrectly

```c
const int arr[] = {1, 2, 3};
arr[0] = 4;             // Compiler error

int *p = arr;           // Error: discards const qualifier
p[0] = 4;               // Also error
```

### 3. Macro Side Effects

```c
#define ABS(x) ((x) < 0 ? -(x) : (x))

int x = -5;
int y = ABS(x++);       // x++ evaluated twice
```

### 4. Forgetting Parentheses in Macros

```c
#define MUL(a, b) a * b

int x = MUL(2 + 3, 4 + 5);   // 2 + 3 * 4 + 5 = 19, not 45
```

### 5. Using `#define` for Constants in Headers

If you define a constant in a header with `#define`, it's fine. If you define `const int` in a header, each translation unit gets its own copy (unless you use `extern`). For true global constants visible across files:

```c
// constants.h
extern const int GLOBAL_MAX;

// constants.c
const int GLOBAL_MAX = 100;
```

---

## Complete Example

```c
#include <stdio.h>

// Preprocessor constants
#define PI 3.14159f
#define MAX(x, y) ((x) > (y) ? (x) : (y))

// Const variables
const float EULER = 2.71828f;
const char *GREETING = "Hello, World!";

// Enum for status codes
enum Status {
    STATUS_OK = 0,
    STATUS_ERROR = 1,
    STATUS_NOT_FOUND = 2,
    STATUS_TIMEOUT = 3
};

// Enum for state machine
typedef enum {
    STATE_IDLE,
    STATE_RUNNING,
    STATE_STOPPED
} State;

void print_status(enum Status s) {
    switch (s) {
        case STATUS_OK:
            printf("OK\n");
            break;
        case STATUS_ERROR:
            printf("ERROR\n");
            break;
        case STATUS_NOT_FOUND:
            printf("NOT FOUND\n");
            break;
        case STATUS_TIMEOUT:
            printf("TIMEOUT\n");
            break;
    }
}

int main(void) {
    printf("PI = %f\n", PI);
    printf("EULER = %f\n", EULER);
    printf("MAX(10, 20) = %d\n", MAX(10, 20));
    
    // Cannot modify const
    // EULER = 3.0;  // Error
    
    // Enum usage
    enum Status s = STATUS_OK;
    print_status(s);
    
    State state = STATE_RUNNING;
    printf("State: %d\n", state);
    
    // String literal (read-only)
    const char *msg = "This is a string literal";
    // msg[0] = 't';  // Undefined behavior
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [05 — Variables](05-variables.md)
- **Next:** [07 — Operators](07-operators.md)
- **Preprocessor details:** [Part 08 — Preprocessor](/08-preprocessor/notes/52-preprocessor.md)
- **Storage class specifiers:** [05 — Variables](05-variables.md)
- **Function-like macros:** [53 — Macros](/08-preprocessor/notes/53-macros.md)
- **Inline functions (C99):** [15 — Inline Functions](/02-functions/notes/15-inline.md)

---

## References

- ISO/IEC 9899:2018 §6.4.4 — Constants
- ISO/IEC 9899:2018 §6.7.3 — Type qualifiers (`const`, `volatile`, `restrict`)
- ISO/IEC 9899:2018 §6.7.2.2 — Enumeration specifiers
- ISO/IEC 9899:2018 §6.10 — Preprocessing directives