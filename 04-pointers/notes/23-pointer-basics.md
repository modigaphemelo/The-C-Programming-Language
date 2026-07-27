# 23: What is a Pointer? — Memory Addresses

---

## Overview

A pointer is a variable that stores a memory address. Instead of holding a value like `5` or `'A'`, a pointer holds the location of a value in memory. This is the fundamental building block of C's power—and its danger.

**Key characteristics:**

- Every variable has a memory address
- A pointer stores that address
- You can access the value at that address (dereferencing)
- Pointers enable dynamic memory, efficient data structures, and system programming

---

## Memory Addresses

When you declare a variable, the compiler allocates memory for it. Every variable has a memory address.

```c
int x = 42;
```

Memory layout (simplified):

```
Address:    0x1000    0x1001    0x1002    0x1003
Value:      0x2A      0x00      0x00      0x00
            [    int x = 42 (little-endian)    ]
```

You can get the address of a variable using the address-of operator `&`.

```c
int x = 42;
printf("%p", (void *)&x);    // Prints the memory address of x
```

---

## Pointer Declaration

```c
type *name;
```

The `*` indicates that the variable is a pointer. The `type` indicates what kind of data the pointer points to.

```c
int *p;          // Pointer to an integer
char *c;         // Pointer to a character
float *f;        // Pointer to a float
void *v;         // Generic pointer (can point to anything)
```

**Important:** A pointer has a type. It knows the size of the data it points to. This affects pointer arithmetic.

---

## Assignment

You assign a pointer by giving it an address.

```c
int x = 42;
int *p = &x;    // p now holds the address of x
```

**Visualizing:**

```
x:  [42]    at address 0x1000
p:  [0x1000]   (points to x)
```

You can also assign one pointer from another:

```c
int *q = p;    // q now also points to x
```

---

## Dereferencing

To access the value at the address stored in a pointer, use the dereference operator `*`.

```c
int x = 42;
int *p = &x;

printf("%d\n", *p);    // 42

*p = 10;               // Changes x to 10
printf("%d\n", x);     // 10
```

**Important:** The `*` has different meanings:

| Context | Meaning |
|---|---|
| `int *p;` | Declaration: p is a pointer to int |
| `*p = 10;` | Dereference: access the value p points to |
| `*p` (in expression) | Dereference: get the value p points to |

---

## Null Pointers

A null pointer points to nothing. It is used to indicate that the pointer is not currently valid.

```c
int *p = NULL;    // p points to nothing
```

**Always check for `NULL` before dereferencing:**

```c
if (p != NULL) {
    *p = 10;
}
```

Dereferencing a null pointer is undefined behavior (and usually a segmentation fault).

```c
int *p = NULL;
*p = 10;    // Segmentation fault (crash)
```

---

## Void Pointers

`void *` is a generic pointer that can point to any type. It is used for functions that need to work with any type of data.

```c
void *p = &x;    // p points to x
```

To use a `void *`, you must cast it to the correct type before dereferencing.

```c
int x = 42;
void *p = &x;
int *q = (int *)p;
printf("%d\n", *q);    // 42
```

---

## Pointers to Pointers

You can have pointers to pointers. This is used for:

- Dynamic 2D arrays
- Modifying pointer arguments in functions
- Linked lists

```c
int x = 42;
int *p = &x;
int **pp = &p;    // pp points to p

printf("%d\n", **pp);    // 42
```

**Visualizing:**

```
x:  [42]
p:  [&x]  → points to x
pp: [&p]  → points to p
```

---

## The Size of Pointers

A pointer is a memory address. The size of a pointer depends on the platform:

- 32-bit: 4 bytes
- 64-bit: 8 bytes

```c
printf("%zu\n", sizeof(int *));    // 4 or 8
printf("%zu\n", sizeof(char *));   // Same as int *
printf("%zu\n", sizeof(void *));   // Same as int *
```

All pointers on a given platform have the same size, regardless of the type they point to.

---

## Common Pitfalls

### 1. Uninitialized Pointers

```c
int *p;        // Uninitialized
*p = 10;       // Undefined behavior
```

### 2. Dereferencing `NULL`

```c
int *p = NULL;
*p = 10;       // Segmentation fault
```

### 3. Forgetting `&` When Assigning

```c
int x = 42;
int *p = x;    // Error: cannot assign int to int *
int *p = &x;   // Correct
```

### 4. Forgetting `*` When Dereferencing

```c
int x = 42;
int *p = &x;
p = 10;        // Changes the pointer, not the value
*p = 10;       // Correct
```

### 5. Returning a Pointer to a Local Variable

```c
int *get_value(void) {
    int x = 42;
    return &x;    // Dangerous: x is destroyed
}
```

### 6. Casting Away `const`

```c
const int x = 42;
int *p = (int *)&x;    // Dangerous
*p = 10;               // Undefined behavior
```

---

## Complete Example

```c
#include <stdio.h>

int main(void) {
    int x = 42;
    int y = 100;
    
    // Basic pointers
    int *p = &x;
    printf("x = %d, *p = %d\n", x, *p);
    
    *p = 10;
    printf("After *p = 10: x = %d\n", x);
    
    // Pointer reassignment
    p = &y;
    printf("After p = &y: *p = %d\n", *p);
    
    // Pointer to pointer
    int **pp = &p;
    printf("**pp = %d\n", **pp);
    
    // Null pointer
    int *null_ptr = NULL;
    if (null_ptr == NULL) {
        printf("null_ptr is NULL\n");
    }
    
    // Void pointer
    void *vp = &x;
    int *qp = (int *)vp;
    printf("qp = %d\n", *qp);
    
    // Sizes
    printf("sizeof(int *) = %zu\n", sizeof(int *));
    printf("sizeof(char *) = %zu\n", sizeof(char *));
    printf("sizeof(void *) = %zu\n", sizeof(void *));
    
    // Address printing
    printf("Address of x: %p\n", (void *)&x);
    printf("Address of p: %p\n", (void *)&p);
    printf("Value of p: %p\n", (void *)p);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [Project 03 — String Manipulation Library](projects/03-string-library/)
- **Next:** [24 — Pointer Operators (`&`, `*`)](24-pointer-operators.md)
- **Pointer arithmetic:** [25 — Pointer Arithmetic](25-pointer-arithmetic.md)
- **Dynamic memory:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)

---

## References

- ISO/IEC 9899:2018 §6.3.2.3 — Pointers
- ISO/IEC 9899:2018 §6.5.3.2 — Address and indirection operators