# 37: Alignment and Padding — Structure Packing

---

## Overview

When you define a structure, the compiler does not simply pack the members together as tightly as possible. It places them at specific offsets to satisfy alignment requirements. This can result in wasted space—padding—between members and at the end of the structure.

Understanding alignment and padding is essential for:

- Controlling memory usage in embedded systems
- Interfacing with hardware
- Creating binary file formats
- Understanding why `sizeof(struct)` is not the sum of `sizeof(members)`

---

## Why Alignment Exists

Processors access memory more efficiently when data is aligned to its natural boundary. For example:

- A 4-byte `int` is most efficiently accessed at an address divisible by 4
- An 8-byte `double` is most efficiently accessed at an address divisible by 8

**Alignment requirements (typical on x86-64):**

| Type | Size | Alignment |
|---|---|---|
| `char` | 1 | 1 |
| `short` | 2 | 2 |
| `int` | 4 | 4 |
| `long` | 8 | 8 |
| `float` | 4 | 4 |
| `double` | 8 | 8 |
| `long double` | 16 | 16 |
| `void *` | 8 | 8 |

---

## Structure Padding Example

```c
struct Example {
    char c;    // 1 byte
    int i;     // 4 bytes
    short s;   // 2 bytes
};
```

**Naive calculation:** 1 + 4 + 2 = 7 bytes

**Actual size on x86-64:** 12 bytes

**Why?**

```
Offset 0: char c (1 byte)
Offset 1-3: padding (3 bytes) to align int
Offset 4-7: int i (4 bytes)
Offset 8-9: short s (2 bytes)
Offset 10-11: padding (2 bytes) to align entire struct
Total: 12 bytes
```

**Visualizing the layout:**

```
+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
|  c  |  P  |  P  |  P  |     i     |     i     |  s  |  s  |  P  |  P  |
+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+-----+
0     1     2     3     4     5     6     7     8     9    10    11    12
```

`P` = padding, `c` = char, `i` = int, `s` = short

---

## Structure Alignment Rules

### Rule 1: Member Alignment

Each member is placed at an offset that is a multiple of its alignment requirement.

```c
struct Example {
    char c;    // offset 0 (multiple of 1)
               // padding 1-3 (3 bytes)
    int i;     // offset 4 (multiple of 4)
    short s;   // offset 8 (multiple of 2)
               // padding 10-11 (2 bytes) to align whole struct
};
```

### Rule 2: Structure Alignment

The alignment of a structure is the maximum alignment of its members.

```c
struct Example {
    char c;    // alignment 1
    int i;     // alignment 4 ← maximum
    short s;   // alignment 2
};
// struct alignment = 4
```

### Rule 3: Structure Size

The size of a structure is a multiple of its alignment.

```c
struct Example {
    char c;    // 1
    int i;     // 4
    short s;   // 2
};
// Size must be multiple of 4 → 12 bytes
```

---

## Optimizing Structure Layout

Order members by size (largest to smallest) to minimize padding.

**Padded (bad):**

```c
struct Bad {
    char c;     // 1 byte
    double d;   // 8 bytes (offset 8, padding 7)
    int i;      // 4 bytes (offset 16, padding 0)
    short s;    // 2 bytes (offset 20, padding 6)
};
// Size: 24 bytes
```

**Optimized (good):**

```c
struct Good {
    double d;   // 8 bytes (offset 0)
    int i;      // 4 bytes (offset 8)
    short s;    // 2 bytes (offset 12)
    char c;     // 1 byte (offset 14)
};
// Size: 16 bytes (padding 1 at end)
```

**Difference:** 24 bytes vs 16 bytes (33% less memory)

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

---

## Controlling Padding

### `#pragma pack` (Compiler-Specific)

Force the compiler to use a specific alignment (GCC, Clang, MSVC).

```c
#pragma pack(push, 1)    // Save current alignment, set to 1

struct Packed {
    char c;     // 1 byte
    int i;      // 4 bytes (offset 1)
    short s;    // 2 bytes (offset 5)
};

#pragma pack(pop)        // Restore previous alignment

// Size: 7 bytes (no padding)
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

// Size: 7 bytes
```

### `__attribute__((aligned(n)))` (GCC/Clang)

Force a structure to be aligned to a specific boundary.

```c
struct Aligned {
    char c;
    int i;
} __attribute__((aligned(16)));

// Size: 16 bytes (minimum)
// Alignment: 16
```

### `_Alignas` (C11)

```c
struct Aligned {
    char c;
    int i;
} _Alignas(16);

// Alignment: 16
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

### 1. Assuming Struct Size = Sum of Members

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

### 3. Misaligned Access on Some Architectures

Some architectures (ARM, SPARC) fault on misaligned access. Packed structures can cause these faults.

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

### 4. Using `offsetof` with Bit Fields

```c
struct Bits {
    unsigned int a : 1;
    unsigned int b : 3;
    unsigned int c : 4;
};

// offsetof cannot be used on bit fields
// offsetof(struct Bits, a);    // Error
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
#include <stdint.h>

struct Padded {
    char c;     // 1 byte, offset 0
    int i;      // 4 bytes, offset 4
    short s;    // 2 bytes, offset 8
};

struct Optimized {
    int i;      // 4 bytes, offset 0
    short s;    // 2 bytes, offset 4
    char c;     // 1 byte, offset 6
};

#pragma pack(push, 1)
struct Packed {
    char c;
    int i;
    short s;
};
#pragma pack(pop)

struct Aligned {
    char c;
    int i;
    short s;
} __attribute__((aligned(16)));

int main(void) {
    printf("=== Padding ===\n");
    printf("sizeof(struct Padded) = %zu\n", sizeof(struct Padded));
    printf("  offsetof(c) = %zu\n", offsetof(struct Padded, c));
    printf("  offsetof(i) = %zu\n", offsetof(struct Padded, i));
    printf("  offsetof(s) = %zu\n", offsetof(struct Padded, s));
    printf("\n");
    
    printf("=== Optimized ===\n");
    printf("sizeof(struct Optimized) = %zu\n", sizeof(struct Optimized));
    printf("  offsetof(i) = %zu\n", offsetof(struct Optimized, i));
    printf("  offsetof(s) = %zu\n", offsetof(struct Optimized, s));
    printf("  offsetof(c) = %zu\n", offsetof(struct Optimized, c));
    printf("\n");
    
    printf("=== Packed ===\n");
    printf("sizeof(struct Packed) = %zu\n", sizeof(struct Packed));
    printf("  offsetof(c) = %zu\n", offsetof(struct Packed, c));
    printf("  offsetof(i) = %zu\n", offsetof(struct Packed, i));
    printf("  offsetof(s) = %zu\n", offsetof(struct Packed, s));
    printf("\n");
    
    printf("=== Aligned ===\n");
    printf("sizeof(struct Aligned) = %zu\n", sizeof(struct Aligned));
    printf("alignment = %zu\n", _Alignof(struct Aligned));
    printf("\n");
    
    // Memory comparison
    struct Padded p1 = {1, 2, 3};
    struct Padded p2 = {1, 2, 3};
    
    // The padding bytes may contain garbage
    // memcmp may fail even though members are equal
    if (memcmp(&p1, &p2, sizeof(p1)) == 0) {
        printf("memcmp: equal\n");
    } else {
        printf("memcmp: not equal (padding bytes differ)\n");
    }
    
    // Correct comparison
    if (p1.c == p2.c && p1.i == p2.i && p1.s == p2.s) {
        printf("Member comparison: equal\n");
    }
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [36 — Memory Layout](36-memory-layout.md)
- **Next:** [Project: Dynamic Array Implementation](projects/05-dynamic-array/)
- **Structures:** [38 — Structures](/06-structures-unions/notes/38-structures.md)
- **Bit fields:** [43 — Bit Fields](/06-structures-unions/notes/43-bit-fields.md)
- **C11 alignas:** Section 6.7.5

---

## References

- ISO/IEC 9899:2018 §6.7.2.1 — Structure and union specifiers
- ISO/IEC 9899:2018 §6.2.8 — Alignment
- GCC: `__attribute__((packed))`, `__attribute__((aligned))`
- MSVC: `#pragma pack`
- `man 3 offsetof` — Offset of structure member