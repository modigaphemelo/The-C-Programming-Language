# 24: Pointer Operators — `&` (Address) and `*` (Dereference)

---

## Overview

Two operators are central to working with pointers:

| Operator | Name | Purpose |
|---|---|---|
| `&` | Address-of | Gets the memory address of a variable |
| `*` | Dereference | Accesses the value at a memory address |

These are the primary tools for working with pointers. Understanding them is essential for using pointers correctly.

---

## The Address-of Operator: `&`

The `&` operator returns the memory address of its operand. It is used to get a pointer to a variable.

```c
int x = 42;
int *p = &x;    // p gets the address of x
```

**What `&` does:**

- Takes a variable as its operand
- Returns the address of that variable
- The type is a pointer to the type of the variable

```c
int x = 42;
printf("%p\n", (void *)&x);    // Prints the address
```

**You can only take the address of an lvalue** (something that has a memory location). You cannot take the address of a literal or a temporary value.

```c
int *p = &42;          // Error: cannot take address of literal
int *p = &(x + y);     // Error: cannot take address of expression
```

---

## The Dereference Operator: `*`

The `*` operator accesses the value at the address stored in a pointer. It is used to read or modify the value a pointer points to.

```c
int x = 42;
int *p = &x;
int y = *p;    // y = 42 (read the value)
*p = 10;       // x = 10 (modify the value)
```

**What `*` does:**

- Takes a pointer as its operand
- Returns the value at the address
- The type is the type the pointer points to

**The `*` operator can be used in two contexts:**

| Context | Meaning |
|---|---|
| `int *p;` | Declaration: p is a pointer to int |
| `*p = 10;` | Dereference: assign 10 to the value p points to |
| `int y = *p;` | Dereference: read the value p points to |

---

## Reading Declarations

The declaration of a pointer can be read from right to left.

```c
int *p;        // p is a pointer to int
char *s;       // s is a pointer to char
int **pp;      // pp is a pointer to a pointer to int
int *arr[10];  // arr is an array of 10 pointers to int
int (*p)[10];  // p is a pointer to an array of 10 ints
int *func();   // func is a function returning a pointer to int
int (*func)(); // func is a pointer to a function returning int
```

**Rule of thumb:** Read right to left, and `*` means "pointer to".

---

## Common Pointer Patterns

### 1. Pointer to a Variable

```c
int x = 42;
int *p = &x;    // p points to x
```

### 2. Pointer to a Pointer

```c
int x = 42;
int *p = &x;
int **pp = &p;    // pp points to p
```

### 3. Pointer to an Array Element

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = &arr[2];    // p points to arr[2] (value 3)
```

### 4. Pointer to a Struct

```c
struct Point {
    int x, y;
};

struct Point pt = {10, 20};
struct Point *p = &pt;
p->x = 30;    // Same as (*p).x = 30
```

### 5. Pointer as a Function Parameter

```c
void increment(int *p) {
    (*p)++;    // Modifies the original value
}
```

---

## Pointer Type Safety

C is weakly typed with pointers. The compiler will warn if you mix pointer types, but it often allows it with a cast.

```c
int x = 42;
char *p = &x;    // Warning: incompatible pointer types
```

**Why this is dangerous:**

```c
int x = 42;
char *p = (char *)&x;
printf("%d\n", *p);    // Only reads the first byte of x
```

The result depends on endianness and is almost certainly not what you intended.

**Recommendation:** Always use the correct pointer type, or use `void *` for generic pointers with explicit casts.

---

## Common Pitfalls

### 1. Dereferencing an Uninitialized Pointer

```c
int *p;
*p = 10;    // Undefined behavior
```

### 2. Dereferencing a Null Pointer

```c
int *p = NULL;
*p = 10;    // Segmentation fault
```

### 3. Forgetting `&` in `scanf`

```c
int x;
scanf("%d", x);    // Error: x is not a pointer
scanf("%d", &x);   // Correct
```

### 4. Confusing `*` in Declaration and Dereference

```c
int *p;    // Declaration: p is a pointer to int
*p = 10;   // Dereference: assign 10 to the value p points to
```

### 5. Taking Address of a Literal

```c
int *p = &42;    // Error
```

### 6. Taking Address of a Register Variable

```c
register int x = 42;
int *p = &x;    // Error
```

---

## Complete Example

```c
#include <stdio.h>

int main(void) {
    // Basic address and dereference
    int x = 42;
    int *p = &x;
    
    printf("x = %d\n", x);
    printf("&x = %p\n", (void *)&x);
    printf("p = %p\n", (void *)p);
    printf("*p = %d\n", *p);
    
    // Modify through pointer
    *p = 10;
    printf("After *p = 10: x = %d\n", x);
    
    // Pointer to pointer
    int **pp = &p;
    printf("pp = %p\n", (void *)pp);
    printf("*pp = %p\n", (void *)*pp);
    printf("**pp = %d\n", **pp);
    
    // Multiple indirection
    ***pp = 100;    // Changes x to 100
    printf("After ***pp = 100: x = %d\n", x);
    
    // Arrays and pointers
    int arr[5] = {1, 2, 3, 4, 5};
    int *arr_ptr = arr;    // arr decays to &arr[0]
    printf("arr[0] = %d, *arr_ptr = %d\n", arr[0], *arr_ptr);
    
    arr_ptr = &arr[2];
    printf("arr[2] = %d, *arr_ptr = %d\n", arr[2], *arr_ptr);
    
    // Function parameter example
    int y = 20;
    void increment(int *q) {
        (*q)++;
    }
    increment(&y);
    printf("After increment: y = %d\n", y);
    
    // Void pointer
    void *vp = &x;
    int *qp = (int *)vp;
    printf("qp = %d\n", *qp);
    
    // sizeof pointers
    printf("\nsizeof(int *) = %zu\n", sizeof(int *));
    printf("sizeof(char *) = %zu\n", sizeof(char *));
    printf("sizeof(void *) = %zu\n", sizeof(void *));
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [23 — What is a Pointer?](23-pointer-basics.md)
- **Next:** [25 — Pointer Arithmetic](25-pointer-arithmetic.md)
- **Pointer parameters:** [27 — Pointer Parameters](27-pointer-parameters.md)
- **Function pointers:** [29 — Function Pointers](29-function-pointers.md)
- **Void pointers:** [30 — Void Pointers](30-void-pointers.md)

---

## References

- ISO/IEC 9899:2018 §6.3.2.3 — Pointers
- ISO/IEC 9899:2018 §6.5.3.2 — Address and indirection operators
- ISO/IEC 9899:2018 §6.7.6.3 — Function declarators
- ISO/IEC 9899:2018 §6.7.6.2 — Array declarators