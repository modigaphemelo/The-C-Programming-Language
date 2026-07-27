# 26: Null Pointers — `NULL`, Dangling Pointers, and Safety

---

## Overview

A null pointer is a pointer that points to nothing. It is used to indicate that a pointer is not currently valid, that a function failed, or that a resource is not available. Dereferencing a null pointer is undefined behavior (and usually causes a segmentation fault). Checking for null pointers before using them is essential for writing robust C code.

---

## What is a Null Pointer?

A null pointer is a pointer with a value of zero (or `NULL`). It does not point to any valid memory location. Attempting to read or write through a null pointer will cause undefined behavior.

```c
int *p = NULL;    // p points to nothing
```

**A null pointer is not the same as an uninitialized pointer.** An uninitialized pointer contains whatever garbage value was in memory. A null pointer has a specific value (0) that indicates it points to nothing.

```c
int *p;           // Uninitialized (garbage)
int *q = NULL;    // Null pointer (known invalid)
```

---

## `NULL` vs `nullptr`

In C, `NULL` is a macro defined in `<stddef.h>`, `<stdio.h>`, `<stdlib.h>`, and others. It is typically defined as:

```c
#define NULL ((void *)0)
```

**Modern C18 uses `NULL`.** There is no `nullptr` in C18—that is a C++ feature (and a C23 feature). Stick with `NULL`.

---

## Dereferencing a Null Pointer

**Never dereference a null pointer.** It causes undefined behavior.

```c
int *p = NULL;
*p = 10;          // Segmentation fault (on most systems)
```

**Always check for `NULL` before dereferencing:**

```c
int *p = get_some_pointer();
if (p != NULL) {
    *p = 10;    // Safe
} else {
    // Handle the error
}
```

---

## Common Uses for Null Pointers

### 1. Indicating Failure

```c
char *s = malloc(100);
if (s == NULL) {
    // Allocation failed
    return -1;
}
```

### 2. Terminating a Linked List

```c
struct Node *head = NULL;    // Empty list
struct Node *p = head;
while (p != NULL) {
    // Process node
    p = p->next;
}
```

### 3. Terminating an Array of Pointers

```c
char *argv[] = {"program", "arg1", "arg2", NULL};
for (int i = 0; argv[i] != NULL; i++) {
    printf("%s\n", argv[i]);
}
```

### 4. Optional Parameters

```c
void print_name(const char *name) {
    if (name == NULL) {
        printf("(no name)\n");
    } else {
        printf("%s\n", name);
    }
}
```

---

## Dangling Pointers

A dangling pointer is a pointer that points to memory that has been freed or is no longer valid. Using a dangling pointer is undefined behavior.

```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);          // p is now dangling
*p = 10;          // Undefined behavior
```

**Common sources of dangling pointers:**

| Source | Description |
|---|---|
| `free()` | Memory is freed but pointer still holds the address |
| Returning local variable address | The variable is destroyed when the function returns |
| Array out of bounds | Pointer points to invalid memory |
| `realloc()` | Old pointer becomes invalid after reallocation |

---

## Setting Pointers to `NULL` After `free`

A good practice is to set a pointer to `NULL` after freeing it.

```c
free(p);
p = NULL;
```

**Why:**
- Prevents accidental use of the freed memory
- Makes it easy to check if the pointer is valid
- Helps debugging

---

## Checking Return Values from `malloc`

Always check the return value of `malloc`, `calloc`, or `realloc`.

```c
int *p = malloc(100 * sizeof(int));
if (p == NULL) {
    // Handle allocation failure
    return -1;
}
// Use p
```

---

## Common Pitfalls

### 1. Dereferencing a Null Pointer

```c
int *p = NULL;
printf("%d\n", *p);    // Segmentation fault
```

### 2. Freeing a Pointer Twice

```c
free(p);
free(p);    // Undefined behavior
```

### 3. Using a Pointer After Free

```c
int *p = malloc(sizeof(int));
free(p);
*p = 10;    // Undefined behavior
```

### 4. Returning a Pointer to a Local Variable

```c
int *get_value(void) {
    int x = 42;
    return &x;    // Dangling pointer
}
```

### 5. Not Checking for `NULL`

```c
int *p = malloc(sizeof(int));
*p = 42;    // If malloc failed, p is NULL
```

---

## Safety Practices

### Rule of Thumb

1. Always initialize pointers to `NULL`
2. Always check `NULL` before dereferencing
3. Set pointers to `NULL` after `free`
4. Never return pointers to local variables
5. Always check `malloc` return values

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

void safe_free(int **p) {
    if (p != NULL && *p != NULL) {
        free(*p);
        *p = NULL;
    }
}

int main(void) {
    // Null pointer
    int *p = NULL;
    if (p == NULL) {
        printf("p is null\n");
    }
    
    // Check before use
    if (p != NULL) {
        *p = 10;    // Not executed
    }
    
    // Allocation with check
    p = malloc(sizeof(int));
    if (p == NULL) {
        printf("Allocation failed\n");
        return 1;
    }
    
    *p = 42;
    printf("*p = %d\n", *p);
    
    // Free and set to NULL
    safe_free(&p);
    
    if (p == NULL) {
        printf("p is null after safe_free\n");
    }
    
    // Dangling pointer prevention
    int *arr = malloc(5 * sizeof(int));
    if (arr == NULL) {
        return 1;
    }
    arr[0] = 10;
    
    free(arr);
    arr = NULL;    // Prevent dangling
    
    // Returning local variable address (dangerous)
    // int *bad = get_local();    // Bad practice
    
    return 0;
}

// This function is dangerous - don't write code like this
// int *get_local(void) {
//     int x = 42;
//     return &x;    // Dangling pointer
// }
```

---

## Cross-References

- **Previous:** [25 — Pointer Arithmetic](25-pointer-arithmetic.md)
- **Next:** [27 — Pointer Parameters](27-pointer-parameters.md)
- **Dynamic memory:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)
- **Memory leaks:** [35 — Memory Leaks](/05-memory/notes/35-memory-leaks.md)

---

## References

- ISO/IEC 9899:2018 §6.3.2.3 — Pointers
- ISO/IEC 9899:2018 §7.22 — Memory management functions `<stdlib.h>`
- ISO/IEC 9899:2018 §7.19 — Common definitions `<stddef.h>` (for `NULL`)