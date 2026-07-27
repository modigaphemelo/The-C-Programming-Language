# 30: Void Pointers — `void*`, Generic Pointers

---

## Overview

A `void *` is a generic pointer. It can point to any type of data, but it cannot be dereferenced directly—you must cast it to the correct type first. This is used for:

- Generic functions (`qsort`, `bsearch`)
- Dynamic memory allocation (`malloc`, `calloc`, `realloc`)
- Callback functions that work with any data type
- Data structures that store arbitrary types

The `void *` is the closest C has to a "generic" type. It is also the most dangerous.

---

## What is a `void *`?

A `void *` is a pointer to an unknown type. It can hold the address of any data type, but you cannot dereference it without casting.

```c
int x = 42;
void *p = &x;    // p points to x, but the type is unknown
```

**Key rules:**

- You cannot dereference a `void *`
- You cannot perform pointer arithmetic on a `void *` (though some compilers allow it as an extension)
- You must cast a `void *` to the correct type before using it

---

## `void *` and `malloc`

The most common use of `void *` is dynamic memory allocation.

```c
#include <stdlib.h>

int *p = malloc(sizeof(int));    // malloc returns void *
*p = 42;                         // Fine: p is int *

char *s = malloc(100);           // malloc returns void *, assigned to char *
strcpy(s, "Hello");
```

`malloc` returns `void *` because it doesn't know what type you want. The assignment to `int *` or `char *` converts the `void *` implicitly (in C). This is legal and common.

---

## Dereferencing a `void *`

You cannot dereference a `void *` directly because the compiler doesn't know the size or type of the data.

```c
int x = 42;
void *p = &x;
printf("%d\n", *p);    // Error: dereferencing void * is not allowed
```

To dereference a `void *`, you must cast it to the correct type.

```c
int x = 42;
void *p = &x;
printf("%d\n", *(int *)p);    // Cast to int * before dereferencing
```

---

## `void *` and Function Parameters

`void *` is used to pass arbitrary data to a function.

### Generic Swap Function

```c
#include <stdio.h>
#include <string.h>

void swap(void *a, void *b, size_t size) {
    char temp[size];
    memcpy(temp, a, size);
    memcpy(a, b, size);
    memcpy(b, temp, size);
}

int main(void) {
    int x = 5, y = 10;
    swap(&x, &y, sizeof(int));
    printf("x = %d, y = %d\n", x, y);
    
    double a = 3.14, b = 2.71;
    swap(&a, &b, sizeof(double));
    printf("a = %f, b = %f\n", a, b);
    
    return 0;
}
```

### Callback with Generic Data

```c
#include <stdio.h>

typedef void (*callback_t)(void *data);

void process(int *arr, size_t n, void *ctx, callback_t cb) {
    for (size_t i = 0; i < n; i++) {
        cb(&arr[i]);    // cb receives a void * to each element
    }
}

void print_int(void *data) {
    int *p = (int *)data;
    printf("%d ", *p);
}

void double_int(void *data) {
    int *p = (int *)data;
    *p *= 2;
}

int main(void) {
    int arr[] = {1, 2, 3, 4, 5};
    size_t n = sizeof(arr) / sizeof(arr[0]);
    
    process(arr, n, NULL, print_int);
    printf("\n");
    
    process(arr, n, NULL, double_int);
    process(arr, n, NULL, print_int);
    printf("\n");
    
    return 0;
}
```

---

## `void *` and Data Structures

`void *` allows data structures to store arbitrary types.

```c
typedef struct {
    void *data;
    size_t size;
    void (*print)(void *);
} GenericValue;

void print_int(void *data) {
    printf("%d", *(int *)data);
}

int main(void) {
    int x = 42;
    GenericValue v = {
        .data = &x,
        .size = sizeof(int),
        .print = print_int
    };
    
    v.print(v.data);    // 42
    printf("\n");
    
    return 0;
}
```

---

## `qsort` with `void *`

The standard library's `qsort` function uses `void *` to work with any array type.

```c
#include <stdlib.h>
#include <stdio.h>

int compare_int(const void *a, const void *b) {
    int x = *(const int *)a;
    int y = *(const int *)b;
    return (x > y) - (x < y);
}

int compare_double(const void *a, const void *b) {
    double x = *(const double *)a;
    double y = *(const double *)b;
    return (x > y) - (x < y);
}

int main(void) {
    int int_arr[] = {5, 2, 8, 1, 9};
    double dbl_arr[] = {3.14, 2.71, 1.41, 4.67};
    
    qsort(int_arr, 5, sizeof(int), compare_int);
    qsort(dbl_arr, 4, sizeof(double), compare_double);
    
    // Print results
    return 0;
}
```

---

## Pointer Arithmetic with `void *`

Standard C does not allow pointer arithmetic on `void *` because the size of the type is unknown.

```c
void *p = malloc(10);
p++;    // Error: arithmetic on void * is not allowed
```

**GCC extension:** Some compilers treat `void *` as `char *` for arithmetic, but this is not portable.

```c
// GCC allows this, but it's not standard:
void *p = malloc(10);
p = (char *)p + 1;    // Portable: cast to char * first
```

---

## Common Pitfalls

### 1. Dereferencing Without Casting

```c
void *p = &x;
printf("%d\n", *p);    // Error: cannot dereference void *
```

### 2. Incorrect Cast

```c
void *p = &x;
double *d = (double *)p;    // Wrong: x is int, not double
printf("%f\n", *d);         // Undefined behavior
```

### 3. Forgetting to Cast in Comparison

```c
void *p = &x;
if (p == &x) {    // Works: comparing addresses
    // ...
}
```

### 4. Using `void *` in Pointer Arithmetic

```c
void *p = malloc(10);
p += 2;    // Error: arithmetic on void * is not allowed
```

### 5. Aliasing Issues with `void *`

```c
int x = 42;
void *p = &x;
double *d = (double *)p;    // Type-punning: undefined behavior
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Generic swap
void swap(void *a, void *b, size_t size) {
    char *temp = malloc(size);
    if (!temp) return;
    
    memcpy(temp, a, size);
    memcpy(a, b, size);
    memcpy(b, temp, size);
    
    free(temp);
}

// Generic print (callback)
typedef void (*print_fn)(void *);

void print_int(void *data) {
    printf("%d", *(int *)data);
}

void print_double(void *data) {
    printf("%f", *(double *)data);
}

void print_string(void *data) {
    printf("%s", (char *)data);
}

// Generic process
void process_array(void *arr, size_t n, size_t size, print_fn fn) {
    char *p = (char *)arr;
    for (size_t i = 0; i < n; i++) {
        fn(p + i * size);
        printf(" ");
    }
    printf("\n");
}

int main(void) {
    // int array
    int int_arr[] = {1, 2, 3, 4, 5};
    size_t int_n = sizeof(int_arr) / sizeof(int_arr[0]);
    
    printf("ints: ");
    process_array(int_arr, int_n, sizeof(int), print_int);
    
    // double array
    double dbl_arr[] = {1.1, 2.2, 3.3, 4.4};
    size_t dbl_n = sizeof(dbl_arr) / sizeof(dbl_arr[0]);
    
    printf("doubles: ");
    process_array(dbl_arr, dbl_n, sizeof(double), print_double);
    
    // string array
    char *str_arr[] = {"one", "two", "three", "four"};
    size_t str_n = sizeof(str_arr) / sizeof(str_arr[0]);
    
    printf("strings: ");
    process_array(str_arr, str_n, sizeof(char *), print_string);
    
    // Generic swap
    int a = 5, b = 10;
    printf("\nBefore swap: a = %d, b = %d\n", a, b);
    swap(&a, &b, sizeof(int));
    printf("After swap: a = %d, b = %d\n", a, b);
    
    // Void pointer with malloc
    void *p = malloc(sizeof(int));
    if (p) {
        *(int *)p = 42;
        printf("\nVoid pointer value: %d\n", *(int *)p);
        free(p);
    }
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [29 — Function Pointers](29-function-pointers.md)
- **Next:** [Project: Pointer Explorer](projects/04-pointer-explorer/)
- **Dynamic memory:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)
- **`qsort`:** `man qsort`
- **`memcpy`:** `man memcpy`

---

## References

- ISO/IEC 9899:2018 §6.3.2.3 — Pointers
- ISO/IEC 9899:2018 §6.5.3.2 — Address and indirection operators
- ISO/IEC 9899:2018 §7.22 — Memory management functions `<stdlib.h>`
- ISO/IEC 9899:2018 §7.24 — String handling `<string.h>`