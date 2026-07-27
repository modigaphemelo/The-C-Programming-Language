# 27: Pointers and Functions — Pass by Pointer

---

## Overview

You already know that C passes arguments by value. This means functions receive copies of arguments, and cannot modify the original variables. To modify variables in the caller's scope, you must pass pointers.

Passing a pointer to a function allows the function to:

- Modify the original value
- Return multiple values
- Work with large data efficiently
- Allocate memory in the caller's scope

---

## Modifying Variables Through Pointers

The most common use of pointer parameters is to modify variables in the caller's scope.

```c
void increment(int *p) {
    (*p)++;    // Modifies the original value
}

int main(void) {
    int x = 5;
    increment(&x);
    printf("%d\n", x);    // 6
    return 0;
}
```

**The key steps:**

1. Pass the address of the variable (`&x`)
2. The function receives a pointer (`int *p`)
3. Dereference the pointer to modify the original (`*p`)

---

## Swapping Values

The classic example is a swap function.

```c
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

int main(void) {
    int x = 5, y = 10;
    swap(&x, &y);
    printf("%d %d\n", x, y);    // 10 5
    return 0;
}
```

**Without pointers, swapping is impossible:**

```c
void swap_bad(int a, int b) {
    int temp = a;
    a = b;
    b = temp;    // Only modifies local copies
}
```

---

## Returning Multiple Values

Functions can only return one value. To return multiple values, pass pointers and modify them.

```c
void get_values(int *x, int *y) {
    *x = 10;
    *y = 20;
}

int main(void) {
    int a, b;
    get_values(&a, &b);
    printf("%d %d\n", a, b);    // 10 20
    return 0;
}
```

---

## Passing Large Structures

Structures can be large. Passing a pointer to a structure is more efficient than passing the structure by value.

```c
struct LargeData {
    int data[1000];
};

void process(struct LargeData *p) {
    // Work with p->data
}

int main(void) {
    struct LargeData my_data;
    process(&my_data);
    return 0;
}
```

**Passing by value would copy the entire structure** (4000 bytes on most systems). Passing a pointer copies only a single address (4 or 8 bytes).

---

## Modifying Strings

Strings are already pointers to characters. Functions can modify them directly.

```c
void to_upper(char *s) {
    while (*s) {
        if (*s >= 'a' && *s <= 'z') {
            *s -= 32;
        }
        s++;
    }
}

int main(void) {
    char str[] = "hello";
    to_upper(str);
    printf("%s\n", str);    // HELLO
    return 0;
}
```

---

## Allocating Memory in Functions

Pass a pointer to a pointer to allocate memory in the caller's scope.

```c
int allocate_buffer(char **buffer, size_t size) {
    *buffer = malloc(size);
    if (*buffer == NULL) {
        return -1;
    }
    return 0;
}

int main(void) {
    char *buf = NULL;
    if (allocate_buffer(&buf, 100) == 0) {
        // Use buf
        free(buf);
    }
    return 0;
}
```

**Why pointer-to-pointer is needed:**

```c
void allocate_bad(char *buffer, size_t size) {
    buffer = malloc(size);    // Only modifies local copy
}

int main(void) {
    char *buf = NULL;
    allocate_bad(buf, 100);
    // buf is still NULL
    return 0;
}
```

---

## Const Pointers in Function Parameters

Use `const` to indicate that a function will not modify the data.

```c
void print_array(const int *arr, size_t size) {
    for (size_t i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}
```

**This tells the caller:** "I won't modify your data." It also helps the compiler optimize.

---

## Common Pitfalls

### 1. Passing the Wrong Type

```c
void func(int *p) { ... }

int main(void) {
    int x = 5;
    func(x);    // Error: cannot pass int to int *
    func(&x);   // Correct
    return 0;
}
```

### 2. Dereferencing a Null Pointer

```c
void func(int *p) {
    *p = 10;    // If p is NULL, this crashes
}

int main(void) {
    int *p = NULL;
    func(p);    // Undefined behavior
    return 0;
}
```

### 3. Forgetting `&` in `scanf`

```c
int x;
scanf("%d", x);    // Error: x is not a pointer
scanf("%d", &x);   // Correct
```

### 4. Returning a Pointer to a Local Variable

```c
int *get_value(void) {
    int x = 42;
    return &x;    // Dangerous: x is destroyed
}
```

### 5. Passing Pointers to Pointers Incorrectly

```c
void allocate(char **p) {
    *p = malloc(10);
}

int main(void) {
    char *buf;
    allocate(buf);    // Error: passing char * where char ** is expected
    allocate(&buf);   // Correct
    return 0;
}
```

---

## Function Pointers as Parameters

You can pass pointers to functions as arguments. This enables callback mechanisms.

```c
void apply(int *arr, size_t size, int (*func)(int)) {
    for (size_t i = 0; i < size; i++) {
        arr[i] = func(arr[i]);
    }
}

int square(int x) {
    return x * x;
}

int main(void) {
    int arr[5] = {1, 2, 3, 4, 5};
    apply(arr, 5, square);
    // arr = {1, 4, 9, 16, 25}
    return 0;
}
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

// Modify variable
void increment(int *p) {
    (*p)++;
}

// Swap
void swap(int *a, int *b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// Return multiple values
void get_values(int *x, int *y) {
    *x = 10;
    *y = 20;
}

// Allocate memory in caller's scope
int allocate_buffer(char **buffer, size_t size) {
    *buffer = malloc(size);
    return (*buffer != NULL) ? 0 : -1;
}

// Process string
void to_upper(char *s) {
    while (*s) {
        if (*s >= 'a' && *s <= 'z') {
            *s -= 32;
        }
        s++;
    }
}

// Function pointer parameter
void apply(int *arr, size_t size, int (*func)(int)) {
    for (size_t i = 0; i < size; i++) {
        arr[i] = func(arr[i]);
    }
}

int square(int x) {
    return x * x;
}

int double_value(int x) {
    return x * 2;
}

int main(void) {
    // Basic pointer parameter
    int x = 5;
    increment(&x);
    printf("After increment: %d\n", x);
    
    // Swap
    int a = 5, b = 10;
    swap(&a, &b);
    printf("After swap: a = %d, b = %d\n", a, b);
    
    // Multiple return values
    int val1, val2;
    get_values(&val1, &val2);
    printf("get_values: %d %d\n", val1, val2);
    
    // String modification
    char str[] = "hello world";
    to_upper(str);
    printf("to_upper: %s\n", str);
    
    // Dynamic allocation
    char *buf;
    if (allocate_buffer(&buf, 100) == 0) {
        strcpy(buf, "Allocated!");
        printf("buf: %s\n", buf);
        free(buf);
    }
    
    // Function pointer
    int arr[5] = {1, 2, 3, 4, 5};
    apply(arr, 5, square);
    printf("After square: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    apply(arr, 5, double_value);
    printf("After double: ");
    for (int i = 0; i < 5; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [26 — Null Pointers (`NULL`)](26-null.md)
- **Next:** [28 — Pointers to Pointers (`**`)](28-pointer-to-pointer.md)
- **Pass by value:** [11 — Pass by Value](11-pass-by-value.md)
- **Function pointers:** [29 — Function Pointers](29-function-pointers.md)
- **Dynamic memory:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)

---

## References

- ISO/IEC 9899:2018 §6.5.3.2 — Address and indirection operators
- ISO/IEC 9899:2018 §6.7.6.3 — Function declarators
- ISO/IEC 9899:2018 §6.5.2.2 — Function calls