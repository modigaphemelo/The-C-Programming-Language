# 39: Structure Padding — Why `sizeof` Lies

---

## Overview

The `sizeof` operator on a structure often returns a value larger than the sum of its members. This is not a bug—it is by design. The compiler adds padding to satisfy alignment requirements.

Understanding why `sizeof` "lies" is essential for:

- Writing memory-efficient code
- Interfacing with hardware registers
- Creating binary file formats
- Sending data over a network
- Avoiding surprising behavior

---

## Why Padding Exists

Processors access memory in chunks. When data is aligned to its natural boundary, the processor can access it in a single cycle. Misaligned access may require multiple cycles or cause a fault.

**Alignment requirements (typical x86-64):**

| Type | Size | Alignment |
|---|---|---|
| `char` | 1 | 1 |
| `short` | 2 | 2 |
| `int` | 4 | 4 |
| `long` | 8 | 8 |
| `float` | 4 | 4 |
| `double` | 8 | 8 |
| `void *` | 8 | 8 |

The compiler adds padding to ensure each member is properly aligned.

---

## The Padding Example

```c
struct Example {
    char c;    // 1 byte
    int i;     // 4 bytes
    short s;   // 2 bytes
};
```

**Why `sizeof` is 12, not 7:**

```
Offset 0:  char c (1 byte)
Offset 1-3: padding (3 bytes) — align int to 4-byte boundary
Offset 4-7: int i (4 bytes)
Offset 8-9: short s (2 bytes)
Offset 10-11: padding (2 bytes) — align entire struct to 4-byte boundary
Total: 12 bytes
```

```
+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
|  c  |  P  |  P  |  P  |     i     |     i     |  s  |  s  |  P  |  P  |
+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
0     1     2     3     4     5     6     7     8     9    10    11    12
```

`P` = padding

---

## Rules of Structure Padding

### Rule 1: Each member is aligned at its natural boundary

The compiler places each member at an offset that is a multiple of its alignment requirement.

```c
struct Example {
    char c;    // alignment 1 → offset 0
    int i;     // alignment 4 → offset must be multiple of 4
    short s;   // alignment 2 → offset must be multiple of 2
};
```

### Rule 2: The structure alignment is the maximum member alignment

```c
struct Example {
    char c;    // alignment 1
    int i;     // alignment 4 ← maximum
    short s;   // alignment 2
};
// Structure alignment = 4
```

### Rule 3: The structure size is a multiple of its alignment

```c
struct Example {
    char c;    // 1
    int i;     // 4
    short s;   // 2
};
// Size must be multiple of 4 → 12 bytes
```

---

## Visualizing Different Layouts

### Bad Layout (Unoptimized)

```c
struct Bad {
    char c;     // 1 byte
    double d;   // 8 bytes (offset 8, padding 7)
    int i;      // 4 bytes (offset 16)
    short s;    // 2 bytes (offset 20, padding 6)
};
// Size: 24 bytes
```

```
+---++---+---++---+---++---+---++---+---++---+---++---+---++---+---+
| c | P | P | P | P | P | P | P |        d        | d | d | i | i | i | i | s | s | P | P | P | P | P | P |
+---++---+---++---+---++---+---++---+---++---+---++---+---++---+---+
0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15  16  17  18  19  20  21  22  23
```

### Optimized Layout

```c
struct Good {
    double d;   // 8 bytes (offset 0)
    int i;      // 4 bytes (offset 8)
    short s;    // 2 bytes (offset 12)
    char c;     // 1 byte (offset 14)
};
// Size: 16 bytes (padding 1 byte at the end)
```

```
+---+---++---+---++---+---++---+---++---+---++---+---++---+---+
|        d        | d | d | i | i | i | i | s | s | c | P |
+---+---++---+---++---+---++---+---++---+---++---+---++---+---+
0   1   2   3   4   5   6   7   8   9  10  11  12  13  14  15
```

**Rule of thumb:** Order members by size, largest to smallest.

---

## The `offsetof` Macro

`offsetof` returns the byte offset of a member within a structure.

```c
#include <stddef.h>

struct Example {
    char c;
    int i;
    short s;
};

printf("offsetof(c) = %zu\n", offsetof(struct Example, c));    // 0
printf("offsetof(i) = %zu\n", offsetof(struct Example, i));    // 4
printf("offsetof(s) = %zu\n", offsetof(struct Example, s));    // 8
```

**Useful for:**
- Understanding padding
- Serialization
- Binary file formats
- Hardware register mapping

---

## Controlling Padding

### `#pragma pack` (Compiler-Specific)

Force the compiler to use a specific alignment.

```c
#pragma pack(push, 1)    // Save current alignment, set to 1

struct Packed {
    char c;     // 1 byte
    int i;      // 4 bytes (offset 1)
    short s;    // 2 bytes (offset 5)
};

#pragma pack(pop)        // Restore previous alignment

// sizeof(struct Packed) = 7 (no padding)
```

**Common packing values:**

| Value | Effect |
|---|---|
| 1 | No padding (packed) |
| 2 | Align to 2-byte boundaries |
| 4 | Align to 4-byte boundaries |
| 8 | Align to 8-byte boundaries |

### `__attribute__((packed))` (GCC/Clang)

```c
struct Packed {
    char c;
    int i;
    short s;
} __attribute__((packed));

// sizeof(struct Packed) = 7
```

### `__attribute__((aligned(n)))` (GCC/Clang)

Force a structure to be aligned to a specific boundary.

```c
struct Aligned {
    char c;
    int i;
} __attribute__((aligned(16)));

// sizeof(struct Aligned) = 16 (minimum)
// alignment = 16
```

### `_Alignas` (C11)

```c
struct Aligned {
    char c;
    int i;
} _Alignas(16);

// alignment = 16
```

---

## Performance vs Memory

| Approach | Memory | Performance |
|---|---|---|
| Default (aligned) | More padding | Faster access |
| Packed (no padding) | Less memory | Slower access (misaligned) |

**When to pack:**

- Embedded systems with limited memory
- Network protocols
- File formats
- Hardware registers

**When NOT to pack:**

- Performance-critical code
- When memory isn't a concern
- When misaligned access causes faults (some architectures)

---

## Common Pitfalls

### 1. Assuming `sizeof = sum of members`

```c
struct Example {
    char c;
    int i;
};

size_t size = sizeof(struct Example);    // 8, not 5
```

### 2. Using `memcmp` on Packed Structures

```c
struct Packed {
    char c;
    int i;
} __attribute__((packed));

struct Packed a = {1, 2};
struct Packed b = {1, 2};

if (memcmp(&a, &b, sizeof(a)) == 0) {
    // May not work correctly with padding
}
```

### 3. Misaligned Access

Some architectures fault on misaligned access. Packed structures can cause these faults.

```c
struct Packed {
    char c;
    int i;
} __attribute__((packed));

struct Packed p = {1, 2};
int *ptr = &p.i;    // May be misaligned
int x = *ptr;       // Fault on some architectures
```

**Fix:** Use `memcpy` to copy values.

```c
int x;
memcpy(&x, &p.i, sizeof(int));
```

### 4. Using `offsetof` on Bit Fields

```c
struct Bits {
    unsigned int a : 1;
    unsigned int b : 3;
};

// offsetof(struct Bits, a);    // Error: cannot use offsetof on bit fields
```

### 5. Platform Differences

Alignment requirements vary between platforms. Code that assumes a specific layout is not portable.

```c
// x86-64: 8 bytes
struct Example {
    char c;
    double d;
};

// ARM: 16 bytes (different alignment rules)
```

---

## Complete Example

```c
#include <stdio.h>
#include <stddef.h>

// Padded (bad layout)
struct Bad {
    char c;
    double d;
    int i;
    short s;
};

// Optimized (good layout)
struct Good {
    double d;
    int i;
    short s;
    char c;
};

// Packed
#pragma pack(push, 1)
struct Packed {
    char c;
    double d;
    int i;
    short s;
};
#pragma pack(pop)

// Aligned
struct Aligned {
    char c;
    int i;
} __attribute__((aligned(16)));

int main(void) {
    printf("=== Bad Layout ===\n");
    printf("sizeof(struct Bad) = %zu\n", sizeof(struct Bad));
    printf("  offsetof(c) = %zu\n", offsetof(struct Bad, c));
    printf("  offsetof(d) = %zu\n", offsetof(struct Bad, d));
    printf("  offsetof(i) = %zu\n", offsetof(struct Bad, i));
    printf("  offsetof(s) = %zu\n", offsetof(struct Bad, s));
    printf("Sum of members = %zu\n",
           sizeof(char) + sizeof(double) + sizeof(int) + sizeof(short));
    printf("\n");
    
    printf("=== Optimized Layout ===\n");
    printf("sizeof(struct Good) = %zu\n", sizeof(struct Good));
    printf("  offsetof(d) = %zu\n", offsetof(struct Good, d));
    printf("  offsetof(i) = %zu\n", offsetof(struct Good, i));
    printf("  offsetof(s) = %zu\n", offsetof(struct Good, s));
    printf("  offsetof(c) = %zu\n", offsetof(struct Good, c));
    printf("\n");
    
    printf("=== Packed ===\n");
    printf("sizeof(struct Packed) = %zu\n", sizeof(struct Packed));
    printf("  offsetof(c) = %zu\n", offsetof(struct Packed, c));
    printf("  offsetof(d) = %zu\n", offsetof(struct Packed, d));
    printf("  offsetof(i) = %zu\n", offsetof(struct Packed, i));
    printf("  offsetof(s) = %zu\n", offsetof(struct Packed, s));
    printf("\n");
    
    printf("=== Aligned ===\n");
    printf("sizeof(struct Aligned) = %zu\n", sizeof(struct Aligned));
    printf("alignment = %zu\n", _Alignof(struct Aligned));
    printf("\n");
    
    // Memory comparison issue
    struct Bad b1 = {'a', 3.14, 42, 10};
    struct Bad b2 = {'a', 3.14, 42, 10};
    
    // Padding bytes may contain garbage
    if (memcmp(&b1, &b2, sizeof(b1)) == 0) {
        printf("memcmp: equal\n");
    } else {
        printf("memcmp: not equal (padding bytes differ)\n");
    }
    
    // Correct comparison
    if (b1.c == b2.c && b1.d == b2.d &&
        b1.i == b2.i && b1.s == b2.s) {
        printf("Member comparison: equal\n");
    }
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [38 — Structures (`struct`)](38-structures.md)
- **Next:** [40 — Pointers to Structures (`->`)](40-struct-pointers.md)
- **Alignment and padding:** [37 — Alignment and Padding](37-alignment.md)
- **Bit fields:** [43 — Bit Fields](43-bit-fields.md)
- **C11 alignas:** Section 6.7.5

---

## References

- ISO/IEC 9899:2018 §6.7.2.1 — Structure and union specifiers
- ISO/IEC 9899:2018 §6.2.8 — Alignment
- GCC: `__attribute__((packed))`, `__attribute__((aligned))`
- MSVC: `#pragma pack`
- `man 3 offsetof` — Offset of structure member