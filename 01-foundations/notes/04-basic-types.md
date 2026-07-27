# 04: Basic Types — `int`, `char`, `float`, `double`

---

## Overview

Every variable in C has a type. The type determines:

- How much memory the variable occupies
- How that memory is interpreted
- Which operations are valid on that variable

C provides a small set of fundamental types. They map directly to hardware—there is no abstraction layer. An `int` is whatever the processor handles most efficiently. A `char` is one byte. A `float` is the machine's floating-point representation.

The types are minimal by design. C assumes you know what you are doing and gives you exactly what the hardware provides.

---

## Integer Types

### Basic Integer Types

| Type | Typical Size | Typical Range |
|---|---|---|
| `char` | 1 byte | -128 to 127 (signed) or 0 to 255 (unsigned) |
| `short` | 2 bytes | -32,768 to 32,767 |
| `int` | 4 bytes | -2,147,483,648 to 2,147,483,647 |
| `long` | 4 or 8 bytes | Platform-dependent |
| `long long` | 8 bytes | -9.2e18 to 9.2e18 |

**Important:** The sizes above are *typical*. The C standard only guarantees minimum ranges:

- `char` — at least 8 bits
- `short` — at least 16 bits
- `int` — at least 16 bits
- `long` — at least 32 bits
- `long long` — at least 64 bits

**Never assume fixed sizes.** Use `<stdint.h>` for exact-width types if you need them.

### Signed and Unsigned

By default, integer types are signed (can represent negative values). Add the `unsigned` keyword to allow only non-negative values, doubling the positive range:

```c
int x = -5;              // Valid
unsigned int y = -5;     // Compiles, but y becomes a large positive number (4294967291 on 32-bit)
```

**Rule of thumb:** Use `int` for general-purpose integers. Use `unsigned` when you specifically need the extra range or are working with bit manipulation. Use `unsigned` for sizes and indices (like array indexing).

### The `char` Type

`char` is a unique type. It is:

- The smallest addressable unit of memory (one byte)
- Guaranteed to be at least 8 bits
- Used to store characters (ASCII values) or small integers

**Character constants:**

```c
char c = 'A';        // ASCII value 65
char nl = '\n';      // Newline (ASCII 10)
char hex = '\x41';   // ASCII 65
```

`char` can be signed or unsigned—the standard does not specify. On x86, `char` is usually signed. On ARM, it can be unsigned. If you need a specific behavior, use `signed char` or `unsigned char` explicitly.

---

## Floating-Point Types

| Type | Typical Size | Precision | Range |
|---|---|---|---|
| `float` | 4 bytes | ~7 decimal digits | ~1.2e-38 to 3.4e38 |
| `double` | 8 bytes | ~15 decimal digits | ~2.3e-308 to 1.7e308 |
| `long double` | 8, 12, or 16 bytes | Platform-dependent | Platform-dependent |

**Recommendation:** Use `double` unless you have a specific reason not to. `float` has limited precision and can introduce rounding errors. `long double` is rarely necessary outside of scientific computing.

**Floating-point representation:**

Floating-point types follow IEEE 754 (on most modern systems). This means:

- Binary representation approximates decimal values
- Not all decimal numbers can be represented exactly (0.1 is a repeating binary fraction)
- Operations are not associative

```c
float f = 0.1f;          // Approximate
double d = 0.1;          // Still approximate, but more precise

// This is surprising to beginners:
if (f == 0.1f) { }       // Likely false
```

---

## Type Sizes: Platform Dependencies

The exact sizes of types depend on the compiler and architecture. This is intentional: C is designed to be efficient on each platform.

**Common 32-bit and 64-bit sizes:**

| Type | 32-bit | 64-bit (Linux/Unix) | 64-bit (Windows) |
|---|---|---|---|
| `char` | 1 | 1 | 1 |
| `short` | 2 | 2 | 2 |
| `int` | 4 | 4 | 4 |
| `long` | 4 | 8 | 4 |
| `long long` | 8 | 8 | 8 |
| `pointer` | 4 | 8 | 8 |

**The Windows/Linux `long` difference** is a common source of portability bugs.

**Check sizes at compile time:**

```c
#include <stdio.h>

int main(void) {
    printf("char:       %zu\n", sizeof(char));
    printf("short:      %zu\n", sizeof(short));
    printf("int:        %zu\n", sizeof(int));
    printf("long:       %zu\n", sizeof(long));
    printf("long long:  %zu\n", sizeof(long long));
    printf("float:      %zu\n", sizeof(float));
    printf("double:     %zu\n", sizeof(double));
    printf("void*:      %zu\n", sizeof(void*));
    return 0;
}
```

`sizeof` returns `size_t` (an unsigned integer type), and `%zu` is the correct format specifier for it.

---

## Exact-Width Types (`<stdint.h>`)

If you need predictable sizes across platforms, use the exact-width types:

```c
#include <stdint.h>

int8_t a;      // Exactly 8 bits, signed
uint8_t b;     // Exactly 8 bits, unsigned
int16_t c;     // Exactly 16 bits, signed
uint32_t d;    // Exactly 32 bits, unsigned
int64_t e;     // Exactly 64 bits, signed
```

**Use these when:**

- Reading/writing binary file formats
- Network protocols
- Embedded systems where register widths matter
- Any situation where the exact size is important

**Also available:**

- `int_leastN_t` — at least N bits (ensures availability)
- `int_fastN_t` — fastest type with at least N bits
- `intptr_t` — integer capable of holding a pointer

---

## Literal Suffixes

Integer and floating-point literals can be suffixed to specify their type:

| Suffix | Type |
|---|---|
| `U` or `u` | `unsigned int` |
| `L` or `l` | `long` |
| `LL` or `ll` | `long long` |
| `UL` or `ul` | `unsigned long` |
| `ULL` or `ull` | `unsigned long long` |
| `F` or `f` | `float` |
| `L` or `l` | `long double` |

**Examples:**

```c
int x = 42;                 // int
unsigned y = 42U;           // unsigned int
long z = 42L;               // long
float f = 3.14F;            // float
double d = 3.14;            // double
long double ld = 3.14L;     // long double
long long ll = 42LL;        // long long
```

---

## Conversion and Promotion

C has rules for implicit conversions. They exist to preserve precision and to allow mixed-type expressions.

### Integer Promotion

In expressions, types smaller than `int` are promoted to `int` (or `unsigned int`). This can produce unexpected behavior:

```c
signed char a = 127;
signed char b = 1;
signed char c = a + b;      // a and b are promoted to int, result is 128 (overflow in signed char)
```

The addition happens at `int` precision, and then the result is truncated back to `signed char`.

### Usual Arithmetic Conversions

When two operands have different types, the "lesser" type is converted to the "greater" type:

`long double` > `double` > `float` > `unsigned long long` > `long long` > `unsigned long` > `long` > `unsigned int` > `int`

**Examples:**

```c
int a = 10;
unsigned int b = 20;
// a is converted to unsigned int for the comparison

int c = 10;
double d = 3.14;
// c is converted to double for the addition
```

### Explicit Conversion (Casting)

You can force a conversion with a cast:

```c
float f = (float)10 / 3;        // Cast prevents integer division
int *p = (int *)0x1000;         // Convert integer to pointer
```

Use casts sparingly. They bypass type safety.

---

## Common Pitfalls

### 1. Assuming Type Sizes

```c
int array[1000];
int size = sizeof(array);          // OK
int bytes = sizeof(array) * 8;     // May overflow if size is large
```

Use `size_t` for sizes and be aware of overflow.

### 2. Signed/Unsigned Mismatch

```c
int a = -1;
unsigned int b = 1;

if (a < b) {
    // May or may not be true — -1 is promoted to unsigned (large positive)
}
```

### 3. Implicit Float-to-Int Truncation

```c
int x = 3.9;        // x becomes 3 (truncated, not rounded)
```

### 4. Comparing Floats for Equality

```c
float a = 0.1 + 0.2;
float b = 0.3;

if (a == b) {       // Almost certainly false
}
```

Compare with an epsilon:

```c
#include <math.h>
if (fabs(a - b) < 1e-6) {    // True
}
```

### 5. `char` Signedness

```c
char c = 200;       // If char is signed, this is implementation-defined
```

Use `unsigned char` when storing binary data.

---

## Complete Example: Type Sizes and Conversions

```c
#include <stdio.h>
#include <stdint.h>
#include <limits.h>
#include <float.h>

int main(void) {
    // Sizes
    printf("int: %zu bytes, range %d to %d\n", 
           sizeof(int), INT_MIN, INT_MAX);
    printf("unsigned int: %zu bytes, range 0 to %u\n",
           sizeof(unsigned int), UINT_MAX);
    
    // Float precision
    printf("float precision: %d digits\n", FLT_DIG);
    printf("double precision: %d digits\n", DBL_DIG);
    
    // Surprising behavior
    float sum = 0.0f;
    for (int i = 0; i < 10; i++) {
        sum += 0.1f;
    }
    printf("10 * 0.1 = %.20f\n", sum);    // Not exactly 1.0
    
    // Exact-width types
    uint32_t my_32bit = 0xFFFFFFFF;
    printf("uint32_t max: %u\n", my_32bit);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [03 — Compilation](03-compilation.md)
- **Next:** [05 — Variables](05-variables.md)
- **Exact-width types:** `man stdint.h`
- **Float limits:** `man float.h`
- **Integer limits:** `man limits.h`
- **Pointer conversions:** See [23 — Pointer Basics](/04-pointers/notes/23-pointer-basics.md)

---

## References

- ISO/IEC 9899:2018 §6.2.5 — Types
- ISO/IEC 9899:2018 §6.3 — Conversions
- ISO/IEC 9899:2018 §7.20 — `<stdint.h>`
- IEEE 754-2008 — Floating-point arithmetic standard