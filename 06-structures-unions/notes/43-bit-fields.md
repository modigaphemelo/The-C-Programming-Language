# 43: Bit Fields — Packing Flags

---

## Overview

Bit fields allow you to specify the exact number of bits to use for a structure member. This is useful for:

- Packing flags into a small space
- Interfacing with hardware registers
- Creating binary file formats
- Network protocols

**Key characteristics:**

- Members can be as small as 1 bit
- The compiler packs them together in memory
- Size and layout are implementation-defined
- Not all types can be used (only `int`, `unsigned int`, `signed int`, and `_Bool` in C99)
- Cannot take the address of a bit field

---

## Declaration

```c
struct Flags {
    unsigned int ready : 1;    // 1 bit (0 or 1)
    unsigned int error : 1;    // 1 bit
    unsigned int busy : 1;     // 1 bit
    unsigned int reserved : 5; // 5 bits (unused)
};
```

**Memory layout (typical):**

```
+-------------------------------------------+
| Flags (4 bytes total)                       |
+---+---+---+-------+--------+--------+-------+
| r | e | b | res   |        |        |       |
| 1 | 1 | 1 | 5     | 24 bits unused        |
+---+---+---+-------+--------+--------+-------+
bit:0 1 2 3-7     8-31
```

---

## Syntax

```c
struct name {
    type member_name : number_of_bits;
};
```

**Rules:**

- Type must be `int`, `unsigned int`, `signed int`, or `_Bool` (C99)
- Number of bits cannot exceed the size of the type
- A bit field of width 0 forces alignment to the next boundary
- Unnamed bit fields are allowed (for padding)

---

## Accessing Bit Fields

Bit fields are accessed like normal structure members.

```c
struct Flags flags;
flags.ready = 1;
flags.error = 0;
flags.busy = 0;

if (flags.ready) {
    printf("Ready!\n");
}
```

---

## Common Use Cases

### 1. Hardware Register Mapping

```c
struct StatusRegister {
    unsigned int ready : 1;
    unsigned int error : 1;
    unsigned int busy : 1;
    unsigned int reserved : 5;
    unsigned int data_ready : 1;
};

struct StatusRegister *reg = (struct StatusRegister *)0x4000;
while (!reg->ready) {
    // Wait
}
```

### 2. Packing Flags

```c
struct FilePermissions {
    unsigned int read : 1;
    unsigned int write : 1;
    unsigned int execute : 1;
    unsigned int reserved : 5;
};
```

### 3. Network Headers

```c
struct IPHeader {
    unsigned int version : 4;
    unsigned int ihl : 4;
    unsigned int tos : 8;
    unsigned int length : 16;
    // ...
};
```

---

## Unnamed Bit Fields

Use unnamed fields for padding.

```c
struct Flags {
    unsigned int flag1 : 1;
    unsigned int : 2;           // 2 bits of padding
    unsigned int flag2 : 1;
    unsigned int : 4;           // 4 bits of padding
    unsigned int flag3 : 1;
};
```

---

## Zero-Length Bit Field

A zero-length bit field forces alignment to the next boundary.

```c
struct Flags {
    unsigned int a : 1;
    unsigned int b : 1;
    unsigned int : 0;           // Force alignment
    unsigned int c : 1;
    unsigned int d : 1;
};
```

---

## Memory Layout and Portability

**Important:** The layout of bit fields is implementation-defined.

| Issue | Description |
|---|---|
| Order | Bits may be packed from LSB to MSB or vice versa |
| Alignment | The compiler may align bit fields differently |
| Endianness | The order of bits depends on the platform |
| Padding | The compiler may add padding between bit fields |

**Example of non-portable code:**

```c
struct Flags {
    unsigned int a : 1;
    unsigned int b : 1;
    unsigned int c : 1;
};

// The memory layout of flags is NOT portable
```

---

## Common Pitfalls

### 1. Reading from a Bit Field

```c
struct Flags f;
f.ready = 1;
f.error = 1;
// The values are stored, but the order is implementation-defined
```

### 2. Using Signed Bit Fields

```c
struct SignedFlags {
    signed int flag : 1;    // Can store -1 or 0 (not 1)
};
```

A signed 1-bit field can only store 0 or -1. Use `unsigned int` for bit fields.

### 3. Taking the Address of a Bit Field

```c
struct Flags f;
int *p = &f.ready;    // Error: cannot take address of bit field
```

### 4. Assuming Size

```c
struct Flags {
    unsigned int a : 1;
    unsigned int b : 1;
    unsigned int c : 1;
};
// sizeof(struct Flags) is NOT necessarily 3 bits
// It is typically 4 bytes on most systems
```

### 5. Overwriting Adjacent Fields

```c
struct Flags f;
f.a = 1;
f.b = 1;
// The compiler may pack a and b into the same byte
// Order and packing are implementation-defined
```

### 6. Using Bit Fields with `memcmp`

```c
struct Flags a = {1, 0, 1};
struct Flags b = {1, 0, 1};
// memcmp may fail due to padding bits
```

---

## Performance Considerations

| Aspect | Bit Field | Integer Flag |
|---|---|---|
| Memory | Compact | More memory |
| Speed | Slower (bit operations) | Faster |
| Portability | Implementation-defined | Portable |
| Readability | Clear | Clear with bit masks |

**Alternative to bit fields:** Use bit masks and macros.

```c
#define FLAG_READ  0x01
#define FLAG_WRITE 0x02
#define FLAG_EXEC  0x04

unsigned int flags = 0;
flags |= FLAG_READ;
flags |= FLAG_WRITE;

if (flags & FLAG_READ) {
    // Read permission
}
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdint.h>

// File permissions
struct FilePermissions {
    unsigned int read : 1;
    unsigned int write : 1;
    unsigned int execute : 1;
    unsigned int reserved : 5;
};

// Hardware status register simulation
struct StatusRegister {
    unsigned int ready : 1;
    unsigned int error : 1;
    unsigned int busy : 1;
    unsigned int data_ready : 1;
    unsigned int reserved : 28;
};

// Network header simulation
struct IPHeader {
    unsigned int version : 4;
    unsigned int ihl : 4;
    unsigned int tos : 8;
    unsigned int length : 16;
};

// With unnamed fields (padding)
struct PackedFlags {
    unsigned int flag1 : 1;
    unsigned int : 2;           // Padding
    unsigned int flag2 : 1;
    unsigned int : 4;           // Padding
    unsigned int flag3 : 1;
};

// Zero-length alignment
struct AlignedFlags {
    unsigned int a : 1;
    unsigned int b : 1;
    unsigned int : 0;           // Force alignment
    unsigned int c : 1;
    unsigned int d : 1;
};

int main(void) {
    // File permissions
    struct FilePermissions perms = {1, 0, 1};
    printf("File permissions: read=%d, write=%d, execute=%d\n",
           perms.read, perms.write, perms.execute);
    printf("Size of FilePermissions: %zu bytes\n", sizeof(perms));
    
    // Modify
    perms.write = 1;
    printf("After modification: write=%d\n", perms.write);
    
    // Status register
    struct StatusRegister status = {1, 0, 1, 0};
    printf("\nStatus: ready=%d, error=%d, busy=%d, data_ready=%d\n",
           status.ready, status.error, status.busy, status.data_ready);
    printf("Size of StatusRegister: %zu bytes\n", sizeof(status));
    
    // IP header
    struct IPHeader ip = {4, 5, 0, 1500};
    printf("\nIP Header: version=%d, ihl=%d, tos=%d, length=%d\n",
           ip.version, ip.ihl, ip.tos, ip.length);
    printf("Size of IPHeader: %zu bytes\n", sizeof(ip));
    
    // Packed flags with padding
    struct PackedFlags pf = {1, 0, 1};
    printf("\nPacked: flag1=%d, flag2=%d, flag3=%d\n",
           pf.flag1, pf.flag2, pf.flag3);
    printf("Size of PackedFlags: %zu bytes\n", sizeof(pf));
    
    // Alignment
    struct AlignedFlags af = {1, 1, 1, 1};
    printf("\nAligned: a=%d, b=%d, c=%d, d=%d\n",
           af.a, af.b, af.c, af.d);
    printf("Size of AlignedFlags: %zu bytes\n", sizeof(af));
    
    // Bit mask alternative
    printf("\nBit mask alternative:\n");
    unsigned int flags = 0;
    flags |= 0x01;  // Read
    flags |= 0x02;  // Write
    flags |= 0x04;  // Execute
    
    if (flags & 0x01) printf("Read enabled\n");
    if (flags & 0x02) printf("Write enabled\n");
    if (flags & 0x04) printf("Execute enabled\n");
    printf("Flags (hex): 0x%02X\n", flags);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [42 — Unions](42-unions.md)
- **Next:** [44 — `typedef`](44-typedef.md)
- **Structures:** [38 — Structures (`struct`)](38-structures.md)
- **Unions:** [42 — Unions](42-unions.md)
- **Alignment and padding:** [37 — Alignment and Padding](37-alignment.md)

---

## References

- ISO/IEC 9899:2018 §6.7.2.1 — Structure and union specifiers
- ISO/IEC 9899:2018 §6.2.6 — Representations of types
- GCC: `__attribute__((packed))`
- MSVC: `#pragma pack`
- `man 3 offsetof` — Offset of structure member