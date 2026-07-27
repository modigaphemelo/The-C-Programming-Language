# 35: Memory Leaks — Detection and Prevention

---

## Overview

A memory leak occurs when you allocate memory on the heap and never free it. The memory remains allocated until the program exits. Over time, leaks accumulate and can cause a program to run out of memory.

**Common causes:**

- Forgetting to call `free`
- Losing the pointer to allocated memory
- Error paths that skip `free`
- Circular references in data structures

---

## Why Memory Leaks Matter

### Short-lived programs

For programs that run briefly, leaks are often less critical—the OS reclaims memory when the program exits.

```c
int main(void) {
    // Short-running program: OS reclaims memory on exit
    int *p = malloc(1000);
    // Forgot free
    return 0;
}
```

### Long-running programs

For programs that run for hours, days, or years, leaks are catastrophic. Memory usage grows until the system crashes.

```c
while (server_is_running) {
    char *buffer = malloc(1024);
    // Process request
    // Forgot free(buffer)
    // Memory leak: 1024 bytes per request
}
```

### Embedded and constrained systems

In systems with limited memory, even small leaks can be fatal.

---

## Common Causes of Memory Leaks

### 1. Forgotten `free`

```c
void func(void) {
    int *p = malloc(sizeof(int));
    // Use p
    // Missing free(p)
}
```

### 2. Losing the Pointer

```c
void func(void) {
    int *p = malloc(100);
    p = malloc(200);    // Original 100 bytes leaked
    free(p);
}
```

### 3. Error Paths

```c
int process(FILE *f) {
    char *buffer = malloc(1024);
    if (buffer == NULL) {
        return -1;    // No leak (malloc failed)
    }
    
    if (f == NULL) {
        return -1;    // LEAK: buffer not freed
    }
    // Use buffer
    free(buffer);
    return 0;
}
```

**Fix:** Free before returning on error paths.

### 4. Data Structures (Linked Lists, Trees)

```c
void create_list(void) {
    struct Node *head = NULL;
    for (int i = 0; i < 100; i++) {
        struct Node *node = malloc(sizeof(struct Node));
        node->value = i;
        node->next = head;
        head = node;
    }
    // No free_list(head) — leak
}
```

### 5. Reassignment Without Free

```c
struct Node *p = malloc(sizeof(struct Node));
p = create_new_node();    // Original node leaked
```

### 6. Threads and Concurrency

Memory allocated in one thread but never freed because the thread exits unexpectedly.

---

## Detection Tools

### Valgrind

The most common tool for detecting memory leaks on Linux.

```bash
valgrind --leak-check=full ./program
```

**Example output:**

```
==12345== 100 bytes in 1 blocks are definitely lost
==12345==    at 0x4C2FB0F: malloc (in /usr/lib/valgrind/vgpreload_memcheck-amd64-linux.so)
==12345==    by 0x40123: main (program.c:10)
```

### Address Sanitizer (ASan)

Modern tool built into GCC and Clang.

```bash
gcc -fsanitize=address -g program.c -o program
./program
```

### Leak Sanitizer (LSan)

Part of ASan, can be used separately.

```bash
gcc -fsanitize=leak -g program.c -o program
./program
```

### Static Analysis

Tools like `clang-tidy` and `cppcheck` can detect some leaks at compile time.

```bash
cppcheck --enable=all program.c
clang-tidy program.c
```

---

## Prevention Strategies

### 1. Always Match `malloc` with `free`

```c
// Pattern:
void *p = malloc(size);
if (p == NULL) {
    // Handle failure
}
// Use p...
free(p);
p = NULL;
```

### 2. Use `goto` for Cleanup (in C)

```c
int process(FILE *f) {
    char *buffer = NULL;
    int *data = NULL;
    int result = -1;
    
    buffer = malloc(1024);
    if (buffer == NULL) {
        goto cleanup;
    }
    
    data = malloc(sizeof(int));
    if (data == NULL) {
        goto cleanup;
    }
    
    // Use buffer and data...
    result = 0;
    
cleanup:
    free(buffer);
    free(data);
    return result;
}
```

### 3. Design for Cleanup

```c
typedef struct {
    int *data;
    size_t size;
} DynamicArray;

void array_init(DynamicArray *arr, size_t size) {
    arr->data = malloc(size * sizeof(int));
    arr->size = size;
}

void array_free(DynamicArray *arr) {
    free(arr->data);
    arr->data = NULL;
    arr->size = 0;
}
```

### 4. Use a Free List

Maintain a list of allocated pointers for cleanup.

### 5. Use Tools Early

Run valgrind and ASan regularly, not just at the end.

---

## Common Pitfalls

### 1. Freeing in the Wrong Order

```c
free(parent);
free(parent->child);    // parent->child is now invalid
```

**Fix:** Free children first.

### 2. Using `realloc` Incorrectly

```c
ptr = realloc(ptr, new_size);    // If realloc fails, ptr is NULL (leak)
```

### 3. Not Freeing on Error Paths

```c
if (error) {
    return -1;    // Leak if memory was allocated
}
```

### 4. Double Free

```c
free(p);
free(p);    // Undefined behavior
```

### 5. Freeing Uninitialized Memory

```c
void *p;
free(p);    // Undefined behavior (p is uninitialized)
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Simple dynamic array
typedef struct {
    int *data;
    size_t size;
    size_t capacity;
} IntArray;

void int_array_init(IntArray *arr) {
    arr->data = NULL;
    arr->size = 0;
    arr->capacity = 0;
}

int int_array_push(IntArray *arr, int value) {
    if (arr->size >= arr->capacity) {
        size_t new_cap = (arr->capacity == 0) ? 4 : arr->capacity * 2;
        int *new_data = realloc(arr->data, new_cap * sizeof(int));
        if (new_data == NULL) {
            return -1;
        }
        arr->data = new_data;
        arr->capacity = new_cap;
    }
    arr->data[arr->size++] = value;
    return 0;
}

void int_array_free(IntArray *arr) {
    free(arr->data);
    arr->data = NULL;
    arr->size = 0;
    arr->capacity = 0;
}

// Function with cleanup using goto
int process_data(const char *filename) {
    FILE *f = NULL;
    char *buffer = NULL;
    IntArray arr;
    int result = -1;
    
    int_array_init(&arr);
    
    f = fopen(filename, "r");
    if (f == NULL) {
        goto cleanup;
    }
    
    buffer = malloc(1024);
    if (buffer == NULL) {
        goto cleanup;
    }
    
    while (fgets(buffer, 1024, f)) {
        int value = atoi(buffer);
        if (int_array_push(&arr, value) != 0) {
            goto cleanup;
        }
    }
    
    // Success
    result = 0;
    
cleanup:
    fclose(f);
    free(buffer);
    int_array_free(&arr);
    return result;
}

int main(void) {
    // Demonstrating a leak (for educational purposes)
    printf("Leaking memory (don't do this):\n");
    for (int i = 0; i < 10; i++) {
        char *s = malloc(100);
        strcpy(s, "Leaked");
        // Leak: s is overwritten each iteration
    }
    printf("Leaked %d allocations\n", 10);
    
    // Correct usage
    IntArray arr;
    int_array_init(&arr);
    
    for (int i = 0; i < 20; i++) {
        int_array_push(&arr, i * i);
    }
    
    printf("Array contents:");
    for (size_t i = 0; i < arr.size; i++) {
        printf(" %d", arr.data[i]);
    }
    printf("\n");
    
    int_array_free(&arr);
    
    // Process data
    if (process_data("test.txt") == 0) {
        printf("Processed data successfully\n");
    }
    
    return 0;
}
```

---

## Leak Detection with Valgrind

```bash
# Compile with debug symbols
gcc -g -std=c18 program.c -o program

# Run valgrind
valgrind --leak-check=full --show-leak-kinds=all ./program

# Output will show:
# - Definitely lost: memory that is definitely leaked
# - Indirectly lost: memory that is reachable only through leaked memory
# - Possibly lost: memory that may be leaked (pointer to interior)
# - Still reachable: memory still allocated at program exit
```

**Interpreting the output:**

```
==12345== 100 bytes in 1 blocks are definitely lost
==12345==    at 0x4C2FB0F: malloc (vg_replace_malloc.c:381)
==12345==    by 0x40123: main (program.c:15)
```

This tells you exactly where the leak occurred.

---

## Cross-References

- **Previous:** [34 — Freeing Memory (`free`)](34-free.md)
- **Next:** [36 — Memory Layout](36-memory-layout.md)
- **Valgrind:** [78 — Valgrind](/12-tools/notes/78-valgrind.md)
- **Address Sanitizer:** [79 — ASan](/12-tools/notes/79-asan.md)
- **Dynamic memory:** [33 — Dynamic Allocation](33-dynamic-allocation.md)

---

## References

- ISO/IEC 9899:2018 §7.22.3 — Memory management functions
- `man valgrind` — Memory debugging tool
- `man asan` — Address Sanitizer
- `man leak` — Leak Sanitizer