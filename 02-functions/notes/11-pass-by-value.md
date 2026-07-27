# 11: Pass by Value — Everything Is a Copy

---

## Overview

C passes arguments to functions **by value**. This means:

- The function receives a copy of the argument's value
- The original value is not modified
- The function operates on its own local copy

This is the most important thing to understand about C functions. Everything else follows from it.

---

## What Pass by Value Means

```c
void increment(int x) {
    x++;      // Modifies the local copy
}

int main(void) {
    int a = 5;
    increment(a);
    printf("%d\n", a);    // Still 5
    return 0;
}
```

The diagram:

```
main:     a = 5
              |
increment: x = 5  (copy)
              |
x++ → x = 6
              |
main:     a = 5  (unchanged)
```

The copy is destroyed when the function returns. The original remains untouched.

---

## Why C Does This

- **Simplicity** — Functions cannot unintentionally modify variables
- **Predictability** — You always know what a function can change
- **Performance** — Copying simple values is cheap

This is why C is called a "pass-by-value" language. Some languages (C++, Python, Java with objects) allow pass-by-reference or pass-by-object-reference. C does not.

---

## What Changes and What Doesn't

### Primitives (int, char, float, etc.)

```c
void set_to_zero(int x) {
    x = 0;
}

int main(void) {
    int a = 42;
    set_to_zero(a);
    printf("%d\n", a);    // 42
    return 0;
}
```

The original value is unchanged.

### Pointers

```c
void set_to_zero(int *p) {
    *p = 0;      // Modifies the thing p points to
}

int main(void) {
    int a = 42;
    set_to_zero(&a);
    printf("%d\n", a);    // 0
    return 0;
}
```

**The pointer itself is passed by value.** The function receives a copy of the pointer. But the copy points to the same memory location, so it can modify what is pointed to.

**Important distinction:**

| Code | Effect |
|---|---|
| `*p = 0;` | Modifies the thing pointed to |
| `p = NULL;` | Modifies the local copy of the pointer |

```c
void set_null(int *p) {
    p = NULL;      // Only modifies local copy
}

int main(void) {
    int a = 42;
    int *p = &a;
    set_null(p);
    printf("%p\n", (void*)p);    // p is still &a, not NULL
    return 0;
}
```

### Arrays

Arrays are passed as pointers to their first element. The pointer is passed by value.

```c
void zero_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        arr[i] = 0;      // Modifies the original array
    }
}

int main(void) {
    int arr[] = {1, 2, 3};
    zero_array(arr, 3);
    // arr is now {0, 0, 0}
    return 0;
}
```

---

## The "Swap" Example

The classic demonstration of pass-by-value's limitation:

```c
void swap_bad(int a, int b) {
    int temp = a;
    a = b;
    b = temp;
}

int main(void) {
    int x = 5, y = 10;
    swap_bad(x, y);
    printf("%d %d\n", x, y);    // Still 5 10
    return 0;
}
```

To swap values, pass pointers:

```c
void swap_good(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void) {
    int x = 5, y = 10;
    swap_good(&x, &y);
    printf("%d %d\n", x, y);    // 10 5
    return 0;
}
```

---

## When to Use Pass by Value vs Pass by Pointer

| Situation | Recommendation |
|---|---|
| Small primitive types (`int`, `char`, `float`) | Pass by value |
| Large structs | Pass by pointer (or const pointer) |
| You need to modify the original | Pass by pointer |
| You need to return multiple values | Pass by pointer |
| You don't want the caller's data modified | Pass by value (or const pointer) |
| Large strings | Pass by pointer (they are arrays anyway) |

---

## Common Pitfalls

### 1. Expecting a Function to Modify the Original

```c
void set_value(int x, int new_value) {
    x = new_value;
}

int main(void) {
    int a = 10;
    set_value(a, 20);
    printf("%d\n", a);    // 10, not 20
}
```

**Fix:** Pass a pointer.

### 2. Modifying the Pointer Itself

```c
void allocate_buffer(char *buffer, int size) {
    buffer = malloc(size);    // Only modifies local copy
}

int main(void) {
    char *buf = NULL;
    allocate_buffer(buf, 100);
    // buf is still NULL
}
```

**Fix:** Pass a pointer to the pointer:

```c
void allocate_buffer(char **buffer, int size) {
    *buffer = malloc(size);
}

int main(void) {
    char *buf = NULL;
    allocate_buffer(&buf, 100);
    // buf is now allocated
}
```

### 3. Returning a Pointer to a Local Variable

```c
int *get_local(void) {
    int x = 42;
    return &x;      // Dangerous: x is destroyed
}
```

**Fix:** Use `static`, dynamic allocation, or pass a pointer.

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

// Pass by value
void increment_value(int x) {
    x++;
    printf("Inside increment_value: x = %d\n", x);
}

// Pass by pointer
void increment_ptr(int *x) {
    (*x)++;
    printf("Inside increment_ptr: *x = %d\n", *x);
}

// Swap (pass by pointer)
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// Modifying the pointer itself (pass by pointer-to-pointer)
void allocate_buffer(char **buffer, int size) {
    *buffer = malloc(size);
    if (*buffer != NULL) {
        (*buffer)[0] = '\0';
    }
}

int main(void) {
    // Pass by value
    int a = 5;
    increment_value(a);
    printf("After increment_value: a = %d\n", a);    // Still 5
    
    // Pass by pointer
    int b = 5;
    increment_ptr(&b);
    printf("After increment_ptr: b = %d\n", b);      // 6
    
    // Swap
    int x = 5, y = 10;
    printf("Before swap: x = %d, y = %d\n", x, y);
    swap(&x, &y);
    printf("After swap: x = %d, y = %d\n", x, y);
    
    // Modifying the pointer
    char *buf = NULL;
    allocate_buffer(&buf, 100);
    if (buf != NULL) {
        printf("Buffer allocated: %p\n", (void*)buf);
        free(buf);
    }
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [10 — Function Syntax](10-function-syntax.md)
- **Next:** [12 — Function Prototypes](12-prototypes.md)
- **Pointer basics:** [23 — What is a Pointer?](/04-pointers/notes/23-pointer-basics.md)
- **Pointer parameters:** [27 — Pointer Parameters](/04-pointers/notes/27-pointer-parameters.md)
- **Dynamic memory:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)

---

## References

- ISO/IEC 9899:2018 §6.5.2.2 — Function calls
- ISO/IEC 9899:2018 §6.9.1 — Function definitions