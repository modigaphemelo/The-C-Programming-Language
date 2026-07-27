# Part 05 Project: Dynamic Array Implementation

---

## Overview

You have learned how memory management works in C—stack vs heap, static allocation, dynamic allocation with `malloc`/`calloc`/`realloc`/`free`, and memory layout. Now you will build a dynamic array library that implements a resizable array using the heap.

This project will require you to:

- Use `malloc`, `calloc`, `realloc`, and `free`
- Handle memory allocation failures
- Track capacity and size
- Implement growth strategies
- Avoid memory leaks
- Use `valgrind` to verify no leaks

This is one of the most useful data structures in C. Understanding dynamic arrays is essential for systems programming.

---

## Project Structure

```
05-memory/projects/05-dynamic-array/
├── README.md
├── dynarray.h
├── dynarray.c
├── test_dynarray.c
├── benchmark.c (bonus)
├── Makefile
└── valgrind.supp (optional)
```

---

## Task 1: Dynamic Array Header

**File:** `dynarray.h`

### Required Interface

```c
#ifndef DYNARRAY_H
#define DYNARRAY_H

#include <stddef.h>

typedef struct {
    int *data;        // Pointer to array data
    size_t size;      // Number of elements currently stored
    size_t capacity;  // Number of elements allocated
} DynArray;

// Initialization
void dynarray_init(DynArray *arr);
void dynarray_free(DynArray *arr);

// Size and capacity
size_t dynarray_size(const DynArray *arr);
size_t dynarray_capacity(const DynArray *arr);
int dynarray_empty(const DynArray *arr);

// Element access
int dynarray_get(const DynArray *arr, size_t index);
void dynarray_set(DynArray *arr, size_t index, int value);

// Modification
int dynarray_push(DynArray *arr, int value);
int dynarray_pop(DynArray *arr);
int dynarray_insert(DynArray *arr, size_t index, int value);
int dynarray_remove(DynArray *arr, size_t index);

// Utility
void dynarray_clear(DynArray *arr);
int dynarray_reserve(DynArray *arr, size_t capacity);
int dynarray_shrink_to_fit(DynArray *arr);

// Iteration
void dynarray_foreach(const DynArray *arr, void (*func)(int *));

// Search
int dynarray_find(const DynArray *arr, int value);

#endif
```

---

## Task 2: Dynamic Array Implementation

**File:** `dynarray.c`

### Required Functions

### `dynarray_init`

Initialize a dynamic array.

```c
void dynarray_init(DynArray *arr) {
    arr->data = NULL;
    arr->size = 0;
    arr->capacity = 0;
}
```

### `dynarray_free`

Free all allocated memory.

```c
void dynarray_free(DynArray *arr) {
    free(arr->data);
    arr->data = NULL;
    arr->size = 0;
    arr->capacity = 0;
}
```

### `dynarray_push`

Add an element to the end. Grow the array if necessary.

```c
int dynarray_push(DynArray *arr, int value) {
    if (arr->size >= arr->capacity) {
        size_t new_cap = (arr->capacity == 0) ? 4 : arr->capacity * 2;
        int *new_data = realloc(arr->data, new_cap * sizeof(int));
        if (new_data == NULL) {
            return -1;  // Allocation failed
        }
        arr->data = new_data;
        arr->capacity = new_cap;
    }
    arr->data[arr->size++] = value;
    return 0;
}
```

### `dynarray_pop`

Remove and return the last element.

```c
int dynarray_pop(DynArray *arr) {
    if (arr->size == 0) {
        return -1;  // Error: empty
    }
    return arr->data[--arr->size];
}
```

### `dynarray_insert`

Insert an element at a specific index.

```c
int dynarray_insert(DynArray *arr, size_t index, int value) {
    if (index > arr->size) {
        return -1;  // Invalid index
    }
    
    if (arr->size >= arr->capacity) {
        size_t new_cap = (arr->capacity == 0) ? 4 : arr->capacity * 2;
        int *new_data = realloc(arr->data, new_cap * sizeof(int));
        if (new_data == NULL) {
            return -1;
        }
        arr->data = new_data;
        arr->capacity = new_cap;
    }
    
    // Shift elements right
    for (size_t i = arr->size; i > index; i--) {
        arr->data[i] = arr->data[i - 1];
    }
    
    arr->data[index] = value;
    arr->size++;
    return 0;
}
```

### `dynarray_remove`

Remove an element at a specific index.

```c
int dynarray_remove(DynArray *arr, size_t index) {
    if (index >= arr->size) {
        return -1;  // Invalid index
    }
    
    int value = arr->data[index];
    
    // Shift elements left
    for (size_t i = index; i < arr->size - 1; i++) {
        arr->data[i] = arr->data[i + 1];
    }
    
    arr->size--;
    return value;
}
```

### `dynarray_shrink_to_fit`

Reduce capacity to match size.

```c
int dynarray_shrink_to_fit(DynArray *arr) {
    if (arr->size == 0) {
        free(arr->data);
        arr->data = NULL;
        arr->capacity = 0;
        return 0;
    }
    
    int *new_data = realloc(arr->data, arr->size * sizeof(int));
    if (new_data == NULL) {
        return -1;
    }
    arr->data = new_data;
    arr->capacity = arr->size;
    return 0;
}
```

### `dynarray_reserve`

Pre-allocate capacity.

```c
int dynarray_reserve(DynArray *arr, size_t capacity) {
    if (capacity <= arr->capacity) {
        return 0;
    }
    
    int *new_data = realloc(arr->data, capacity * sizeof(int));
    if (new_data == NULL) {
        return -1;
    }
    arr->data = new_data;
    arr->capacity = capacity;
    return 0;
}
```

### `dynarray_get`

Get element at index (with bounds checking).

```c
int dynarray_get(const DynArray *arr, size_t index) {
    if (index >= arr->size) {
        return -1;  // Error: out of bounds
    }
    return arr->data[index];
}
```

### `dynarray_set`

Set element at index (with bounds checking).

```c
void dynarray_set(DynArray *arr, size_t index, int value) {
    if (index < arr->size) {
        arr->data[index] = value;
    }
}
```

### `dynarray_find`

Find first occurrence of a value.

```c
int dynarray_find(const DynArray *arr, int value) {
    for (size_t i = 0; i < arr->size; i++) {
        if (arr->data[i] == value) {
            return (int)i;
        }
    }
    return -1;
}
```

### `dynarray_foreach`

Apply a function to each element.

```c
void dynarray_foreach(const DynArray *arr, void (*func)(int *)) {
    for (size_t i = 0; i < arr->size; i++) {
        func(&arr->data[i]);
    }
}
```

---

## Task 3: Test Program

**File:** `test_dynarray.c`

### Required Tests

1. **Basic Operations**
   - Initialize, push, pop, size, capacity
   - Get, set
   - Empty check

2. **Growth**
   - Push beyond initial capacity
   - Verify capacity grows
   - Verify data is preserved

3. **Insert and Remove**
   - Insert at beginning, middle, end
   - Remove from beginning, middle, end
   - Verify data is preserved

4. **Edge Cases**
   - Push to empty array
   - Pop from empty array
   - Insert at invalid index
   - Get from invalid index

5. **Memory**
   - No memory leaks (verify with valgrind)
   - Shrink to fit
   - Reserve capacity

6. **Performance**
   - Push 100,000 elements
   - Verify growth pattern

### Test Output

```
=== DynArray Tests ===

Basic Operations:
  Initial: size=0, capacity=0
  Push 1-5: size=5, capacity=8
  Pop: 5, size=4
  Get(2): 3
  Set(2, 99): 99
  Find(3): -1
  Find(99): 2

Growth:
  Pushing 20 elements...
  Capacity growth: 0->4->8->16->32 (20 elements)

Insert/Remove:
  Insert 99 at 0: [99,1,2,3,4]
  Remove at 0: 99, [1,2,3,4]
  Insert 99 at end: [1,2,3,4,99]
  Remove at end: 99, [1,2,3,4]

Edge Cases:
  Pop from empty: -1
  Get from invalid index: -1
  Insert at invalid index: -1

Shrink to Fit:
  Before: size=4, capacity=8
  After shrink: size=4, capacity=4

All tests passed!
No memory leaks detected.
```

---

## Task 4: Makefile

```makefile
CC = gcc
CFLAGS = -std=c18 -Wall -Wextra -Werror -I.
TARGET = test_dynarray
BENCHMARK = benchmark
OBJS = dynarray.o test_dynarray.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

benchmark: benchmark.c dynarray.c
	$(CC) $(CFLAGS) -o $(BENCHMARK) benchmark.c dynarray.c

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET) $(BENCHMARK)

test: $(TARGET)
	./$(TARGET)

valgrind: $(TARGET)
	valgrind --leak-check=full --show-leak-kinds=all ./$(TARGET)

.PHONY: all clean test valgrind
```

---

## Task 5: Bonus — Benchmark

**File:** `benchmark.c`

Compare dynamic array performance with standard arrays.

```c
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
#include "dynarray.h"

#define TEST_SIZE 1000000

void benchmark_dynarray(void) {
    DynArray arr;
    dynarray_init(&arr);
    
    clock_t start = clock();
    for (int i = 0; i < TEST_SIZE; i++) {
        dynarray_push(&arr, i);
    }
    clock_t end = clock();
    
    double time = (double)(end - start) / CLOCKS_PER_SEC;
    printf("DynArray: pushed %d elements in %.3f seconds\n", TEST_SIZE, time);
    printf("  Final capacity: %zu\n", arr.capacity);
    
    dynarray_free(&arr);
}

void benchmark_std_array(void) {
    int *arr = malloc(TEST_SIZE * sizeof(int));
    if (arr == NULL) {
        printf("malloc failed\n");
        return;
    }
    
    clock_t start = clock();
    for (int i = 0; i < TEST_SIZE; i++) {
        arr[i] = i;
    }
    clock_t end = clock();
    
    double time = (double)(end - start) / CLOCKS_PER_SEC;
    printf("Standard array: filled %d elements in %.3f seconds\n", TEST_SIZE, time);
    
    free(arr);
}

int main(void) {
    benchmark_std_array();
    benchmark_dynarray();
    return 0;
}
```

---

## Checklist

| Task | File | Completed |
|---|---|---|
| Header | `dynarray.h` | [ ] |
| Implementation | `dynarray.c` | [ ] |
| Tests | `test_dynarray.c` | [ ] |
| Makefile | `Makefile` | [ ] |
| README | `README.md` | [ ] |
| Benchmark | `benchmark.c` | [ ] |
| Valgrind verification | — | [ ] |

---

## Implementation Notes

### Growth Strategy

The classic doubling strategy:

```c
size_t new_cap = (arr->capacity == 0) ? 4 : arr->capacity * 2;
```

This gives amortized O(1) push operations.

### Memory Safety

Always check `malloc`/`realloc` return values:

```c
int *new_data = realloc(arr->data, new_cap * sizeof(int));
if (new_data == NULL) {
    return -1;  // Original array is still valid
}
arr->data = new_data;
```

### Shrink Strategy

Consider shrinking when `size < capacity / 4` to avoid thrashing.

### Error Handling

Return `-1` or `NULL` for errors. This is simple and sufficient for this project.

---

## Checking Your Work

### Compile with warnings

```bash
make
```

### Run tests

```bash
make test
```

### Check for memory leaks

```bash
make valgrind
```

### Run benchmark

```bash
make benchmark
./benchmark
```

---

## What You've Learned

After completing this project, you have demonstrated:

- Using `malloc`, `calloc`, `realloc`, and `free`
- Managing dynamic memory correctly
- Implementing a resizable array
- Handling memory allocation failures
- Using `valgrind` to detect memory leaks
- Understanding growth strategies
- Separating interface from implementation

You have built a useful data structure from scratch. This is a significant milestone.

---

**Next:** [Part 06: Structures and Unions](/06-structures-unions/notes/38-structures.md)