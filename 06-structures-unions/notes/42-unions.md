# 42: Unions — Overlapping Memory

---

## Overview

A union is a special data type that allows storing different data types in the same memory location. Unlike a structure, where each member has its own memory, all members of a union share the same memory. The size of a union is determined by its largest member.

**Key characteristics:**

- All members share the same memory location
- Only one member can be active at a time
- Size is determined by the largest member
- Writing to one member overwrites the others
- Useful for memory efficiency and type punning

---

## Declaration

```c
union Data {
    int i;
    float f;
    char str[20];
};
```

**Memory layout:**

```
+------------------+
|      Union       |
|  (largest member)|
|   size = 20      |
+------------------+
| int i   |         |
| float f |         |
| char str[20]     |
+------------------+
```

All members share the same 20 bytes. Only one is valid at a time.

---

## Accessing Union Members

```c
union Data data;
data.i = 42;
printf("%d\n", data.i);    // 42

data.f = 3.14;
printf("%f\n", data.f);    // 3.14
printf("%d\n", data.i);    // Garbage (data.i was overwritten)
```

**Important:** Writing to one member overwrites the others.

---

## Union vs Structure

| Aspect | Union | Structure |
|---|---|---|
| Memory | Shared (overlapping) | Separate (non-overlapping) |
| Size | Largest member | Sum of members (+ padding) |
| Valid members | One at a time | All at once |
| Use case | Memory efficiency, type punning | Grouping related data |

```c
struct S {
    int i;
    float f;
    char c;
};  // sizeof(S) ≈ 12 (or more)

union U {
    int i;
    float f;
    char c;
};  // sizeof(U) = 4 (largest member)
```

---

## Common Use Cases

### 1. Memory Efficiency

When you need to store different types at different times, a union saves memory.

```c
typedef struct {
    int type;  // 0 = int, 1 = float, 2 = string
    union {
        int i;
        float f;
        char str[100];
    } value;
} Variant;

Variant v;
v.type = 0;
v.value.i = 42;

v.type = 1;
v.value.f = 3.14;
```

### 2. Type Punning

Reinterpreting the bits of one type as another. This is platform-dependent and sometimes undefined behavior.

```c
union FloatBits {
    float f;
    uint32_t bits;
};

union FloatBits fb;
fb.f = 3.14;
printf("0x%08X\n", fb.bits);    // Print hex representation of float
```

### 3. Hardware Registers

Accessing hardware registers as both a whole and as individual bits.

```c
union StatusReg {
    uint32_t value;
    struct {
        unsigned int ready : 1;
        unsigned int error : 1;
        unsigned int busy : 1;
        unsigned int reserved : 29;
    } bits;
};
```

### 4. IP Address Representation

```c
union IPAddress {
    uint32_t address;
    uint8_t octets[4];
};

union IPAddress ip;
ip.address = 0x01020304;    // 1.2.3.4
printf("%d.%d.%d.%d\n", ip.octets[0], ip.octets[1], ip.octets[2], ip.octets[3]);
```

---

## Anonymous Unions (C11)

C11 introduced anonymous unions, which allow members to be accessed without a name.

```c
typedef struct {
    int type;
    union {
        int i;
        float f;
        char *s;
    };    // No name — members are accessed directly
} Variant;

Variant v;
v.type = 0;
v.i = 42;    // Direct access (no .value)
```

---

## Initialization

```c
union Data d1 = {42};           // Initializes first member (int)
union Data d2 = {.f = 3.14};    // Designated initializer (C99)
union Data d3 = {.str = "Hello"}; // Designated initializer
```

---

## Common Pitfalls

### 1. Reading the Wrong Member

```c
union Data d;
d.i = 42;
printf("%f\n", d.f);    // Garbage (reading wrong member)
```

### 2. Type Punning and Strict Aliasing

Type punning through unions can be undefined behavior due to strict aliasing rules.

```c
union {
    int i;
    float f;
} u;

u.i = 42;
float x = u.f;    // May be undefined behavior
```

**Portable alternative:** Use `memcpy`.

```c
int i = 42;
float f;
memcpy(&f, &i, sizeof(float));    // Copies the bits
```

### 3. Assuming Size

```c
union U {
    int i;
    float f;
    char c;
};

size_t size = sizeof(union U);    // 4 (int) or 4 (float)
```

### 4. Nested Unions

```c
union Outer {
    int i;
    union Inner {
        float f;
        char c;
    } inner;
};
```

### 5. Using Unions with Pointers

```c
union Data {
    int i;
    float f;
};

union Data d;
int *p = &d.i;      // OK
float *q = &d.f;    // OK
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdint.h>
#include <string.h>

// Basic union
union Data {
    int i;
    float f;
    char str[20];
};

// Variant type (using anonymous union)
typedef struct {
    int type;  // 0 = int, 1 = float, 2 = string
    union {
        int i;
        float f;
        char *s;
    };
} Variant;

// IP address
union IPAddress {
    uint32_t address;
    uint8_t octets[4];
};

// Float bits (type punning)
union FloatBits {
    float f;
    uint32_t bits;
};

void print_variant(const Variant *v) {
    switch (v->type) {
        case 0:
            printf("int: %d\n", v->i);
            break;
        case 1:
            printf("float: %f\n", v->f);
            break;
        case 2:
            printf("string: %s\n", v->s);
            break;
        default:
            printf("unknown\n");
    }
}

int main(void) {
    // Basic union
    union Data d;
    d.i = 42;
    printf("d.i = %d\n", d.i);
    
    d.f = 3.14;
    printf("d.f = %f\n", d.f);
    printf("d.i = %d (overwritten)\n", d.i);
    
    strcpy(d.str, "Hello");
    printf("d.str = %s\n", d.str);
    printf("d.i = %d (overwritten)\n", d.i);
    
    // Size
    printf("\nsizeof(union Data) = %zu\n", sizeof(union Data));
    printf("sizeof(int) = %zu, sizeof(float) = %zu, sizeof(str) = %zu\n",
           sizeof(int), sizeof(float), sizeof(char[20]));
    
    // Variant type
    Variant v1 = {0, .i = 42};
    Variant v2 = {1, .f = 3.14};
    Variant v3 = {2, .s = "Hello, World!"};
    
    printf("\nVariants:\n");
    print_variant(&v1);
    print_variant(&v2);
    print_variant(&v3);
    
    // IP address
    union IPAddress ip;
    ip.address = 0x01020304;    // 1.2.3.4 in network order (big-endian)
    printf("\nIP Address: %d.%d.%d.%d\n",
           ip.octets[0], ip.octets[1], ip.octets[2], ip.octets[3]);
    
    // Float bits
    union FloatBits fb;
    fb.f = 3.14159f;
    printf("\nFloat: %f, Hex: 0x%08X\n", fb.f, fb.bits);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [41 — Nested Structures](41-nested-structs.md)
- **Next:** [43 — Bit Fields](43-bit-fields.md)
- **Structures:** [38 — Structures (`struct`)](38-structures.md)
- **Bit fields:** [43 — Bit Fields](43-bit-fields.md)
- **Typedef:** [44 — `typedef`](44-typedef.md)
- **C11 unions:** §6.7.2.1

---

## References

- ISO/IEC 9899:2018 §6.7.2.1 — Structure and union specifiers
- ISO/IEC 9899:2018 §6.7.9 — Initialization
- ISO/IEC 9899:2018 §6.5.2.3 — Structure and union members
- C11: Anonymous unions (Section 6.7.2.1)