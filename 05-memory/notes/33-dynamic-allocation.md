# 33: Dynamic Allocation — `malloc`, `calloc`, `realloc`

---

## Overview

Dynamic allocation is memory that is allocated at runtime, on the heap. You control when it is created and when it is destroyed. This gives you flexibility that static and automatic allocation cannot provide.

**Key characteristics:**

- Allocated at runtime (by you)
- Lifetime is controlled by you (until you call `free`)
- Size can be determined at runtime
- Memory comes from the heap
- Must be freed explicitly to avoid leaks

---

## The Memory Management Functions

All dynamic memory functions are in `<stdlib.h>`.

| Function | Purpose |
|---|---|
| `malloc(size_t size)` | Allocates `size` bytes of uninitialized memory |
| `calloc(size_t count, size_t size)` | Allocates `count * size` bytes, zero-initialized |
| `realloc(void *ptr, size_t new_size)` | Resizes an existing allocation |
| `free(void *ptr)` | Deallocates memory |

---

## `malloc` — Memory Allocation

```c
void *malloc(size_t size);
```

Allocates `size` bytes of uninitialized memory. Returns a pointer to the allocated memory, or `NULL` on failure.

**Example:**

```c
int *p = malloc(sizeof(int));
if (p == NULL) {
    // Handle allocation failure
    return -1;
}
*p = 42;
```

**Allocating an array:**

```c
int *arr = malloc(10 * sizeof(int));
if (arr == NULL) {
    return -1;
}
for (int i = 0; i < 10; i++) {
    arr[i] = i;
}
```

**Important:** `malloc` does not initialize the memory. The contents are garbage.

---

## `calloc` — Contiguous Allocation

```c
void *calloc(size_t count, size_t size);
```

Allocates `count * size` bytes and initializes all bytes to zero.

**Example:**

```c
int *p = calloc(10, sizeof(int));
if (p == NULL) {
    return -1;
}
// All 10 elements are zero-initialized
```

**When to use `calloc` vs `malloc`:**

| Situation | Recommendation |
|---|---|
| Need zero-initialized memory | Use `calloc` |
| Need to allocate and fill immediately | Use `malloc` |
| Security-sensitive data | Use `calloc` (previons data not exposed) |
| Large allocations | `calloc` may be slower (zeroing takes time) |

---

## `realloc` — Resizing Memory

```c
void *realloc(void *ptr, size_t new_size);
```

Resizes an existing allocation. If `new_size` is larger, the additional memory is uninitialized. If `new_size` is smaller, the data is truncated.

**Important:** `realloc` may move the memory to a new location. Always use the returned pointer.

```c
int *arr = malloc(5 * sizeof(int));
if (arr == NULL) {
    return -1;
}
// Fill arr...

// Resize to 10 elements
int *temp = realloc(arr, 10 * sizeof(int));
if (temp == NULL) {
    // realloc failed, original memory is still valid
    // Handle error
    free(arr);
    return -1;
}
arr = temp;
```

**Common pitfall:**

```c
arr = realloc(arr, new_size);    // If realloc fails, arr is NULL (leak!)
```

**Always use a temporary pointer:**

```c
int *temp = realloc(arr, new_size);
if (temp == NULL) {
    // Handle error, arr is still valid
    return -1;
}
arr = temp;
```

---

## `free` — Deallocating Memory

```c
void free(void *ptr);
```

Deallocates memory previously allocated by `malloc`, `calloc`, or `realloc`. The pointer becomes invalid (dangling) after `free`.

**Rule:** Every `malloc`, `calloc`, or `realloc` must have a matching `free`.

```c
int *p = malloc(sizeof(int));
if (p) {
    *p = 42;
    // ...
    free(p);    // Free when done
    p = NULL;   // Prevent dangling pointer
}
```

**What happens if you don't free?**

- Memory leak (program uses more memory over time)
- Eventually, the program may run out of memory
- The OS reclaims memory when the program exits

---

## Common Patterns

### 1. Allocating a Structure

```c
struct Point {
    int x, y;
};

struct Point *p = malloc(sizeof(struct Point));
if (p) {
    p->x = 10;
    p->y = 20;
    // ...
    free(p);
}
```

### 2. Allocating a String

```c
char *s = malloc(100 * sizeof(char));
if (s) {
    strcpy(s, "Hello");
    // ...
    free(s);
}
```

### 3. Allocating a 2D Array

```c
int **matrix = malloc(rows * sizeof(int *));
if (matrix) {
    for (int i = 0; i < rows; i++) {
        matrix[i] = malloc(cols * sizeof(int));
        if (matrix[i] == NULL) {
            // Handle error, free previous rows
            for (int j = 0; j < i; j++) {
                free(matrix[j]);
            }
            free(matrix);
            return NULL;
        }
    }
}
```

### 4. Growing an Array (Dynamic Array)

```c
int *arr = NULL;
int capacity = 0;
int size = 0;

void append(int value) {
    if (size >= capacity) {
        capacity = (capacity == 0) ? 4 : capacity * 2;
        int *temp = realloc(arr, capacity * sizeof(int));
        if (temp == NULL) {
            return;  // Allocation failed
        }
        arr = temp;
    }
    arr[size++] = value;
}
```

---

## Common Pitfalls

### 1. Memory Leak

```c
void func(void) {
    int *p = malloc(sizeof(int));
    // Forgot to free(p)
}
```

### 2. Double Free

```c
free(p);
free(p);    // Undefined behavior
```

### 3. Using Memory After Free

```c
free(p);
*p = 10;    // Undefined behavior
```

### 4. Not Checking `malloc` Return Value

```c
int *p = malloc(sizeof(int));
*p = 42;    // If malloc fails, p is NULL
```

### 5. Using `realloc` Incorrectly

```c
arr = realloc(arr, new_size);    // If realloc fails, arr is NULL (leak)
```

### 6. Allocating the Wrong Size

```c
int *p = malloc(10);    // Allocates 10 bytes, not 10 ints
int *p = malloc(10 * sizeof(int));    // Correct
```

### 7. Using Uninitialized Memory

```c
int *p = malloc(sizeof(int));
printf("%d\n", *p);    // Garbage value
```

### 8. Freeing Stack Memory

```c
int x = 42;
free(&x);    // Error: stack memory cannot be freed
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(void) {
    // malloc
    int *p = malloc(sizeof(int));
    if (p == NULL) {
        printf("malloc failed\n");
        return 1;
    }
    *p = 42;
    printf("*p = %d\n", *p);
    
    // calloc
    int *arr = calloc(5, sizeof(int));
    if (arr == NULL) {
        printf("calloc failed\n");
        free(p);
        return 1;
    }
    printf("calloc arr[0] = %d (zero-initialized)\n", arr[0]);
    
    // realloc
    int *temp = realloc(arr, 10 * sizeof(int));
    if (temp == NULL) {
        printf("realloc failed\n");
        free(arr);
        free(p);
        return 1;
    }
    arr = temp;
    printf("realloc succeeded (arr[0] still %d)\n", arr[0]);
    
    // Free
    free(p);
    free(arr);
    
    // Allocating a string
    char *s = malloc(20 * sizeof(char));
    if (s) {
        strcpy(s, "Hello, World!");
        printf("String: %s\n", s);
        free(s);
    }
    
    // Allocating a structure
    struct Point {
        int x, y;
    };
    
    struct Point *pt = malloc(sizeof(struct Point));
    if (pt) {
        pt->x = 10;
        pt->y = 20;
        printf("Point: (%d, %d)\n", pt->x, pt->y);
        free(pt);
    }
    
    // Dynamic array (growing)
    int *dyn_arr = NULL;
    int capacity = 0;
    int size = 0;
    
    for (int i = 0; i < 10; i++) {
        if (size >= capacity) {
            capacity = (capacity == 0) ? 4 : capacity * 2;
            int *new_arr = realloc(dyn_arr, capacity * sizeof(int));
            if (new_arr == NULL) {
                printf("realloc failed at i = %d\n", i);
                free(dyn_arr);
                return 1;
            }
            dyn_arr = new_arr;
        }
        dyn_arr[size++] = i * i;
    }
    
    printf("Dynamic array:");
    for (int i = 0; i < size; i++) {
        printf(" %d", dyn_arr[i]);
    }
    printf("\n");
    
    free(dyn_arr);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [32 — Static Allocation](32-static-allocation.md)
- **Next:** [34 — Freeing Memory (`free`)](34-free.md)
- **Memory leaks:** [35 — Memory Leaks](35-memory-leaks.md)
- **Null pointers:** [26 — Null Pointers](26-null.md)

---

## References

- ISO/IEC 9899:2018 §7.22.3 — Memory management functions `<stdlib.h>`
- `man malloc` — Memory allocation functions