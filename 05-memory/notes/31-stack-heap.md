# 31: Stack vs Heap — Lifetime and Size Differences

---

## Overview

Memory in a C program is divided into several regions. The two most important for the programmer are the **stack** and the **heap**. Understanding the difference is essential for writing correct C code.

**Key differences:**

| Aspect | Stack | Heap |
|---|---|---|
| Allocation | Automatic (compiler) | Manual (you call `malloc`) |
| Lifetime | Function scope | Until you call `free` |
| Size | Fixed (small) | Flexible (large) |
| Speed | Fast | Slow |
| Management | Automatic | Manual |
| Fragmentation | None | Possible |

---

## The Stack

The stack is a region of memory that grows and shrinks automatically as functions are called and return. It is used for:

- Local variables (non-static)
- Function parameters
- Return addresses

**Characteristics:**

- **Automatic allocation:** Variables are created when the function is entered and destroyed when it returns.
- **LIFO (Last In, First Out):** Functions are called in a stack-like fashion.
- **Fixed size:** The stack size is set at compile time (usually 1-8 MB).
- **Fast:** Allocating and deallocating is just moving a pointer.

**Example:**

```c
void function(void) {
    int x = 42;        // Allocated on the stack
    char buffer[100];  // Allocated on the stack
    // x and buffer are destroyed when function returns
}
```

**Stack overflow:** If you allocate too much memory on the stack (e.g., a large array), you can overflow the stack and cause a segmentation fault.

```c
void bad(void) {
    int arr[1000000];    // May cause stack overflow
}
```

**Recursion and the stack:** Each recursive call adds a new frame to the stack. Deep recursion can cause stack overflow.

```c
void deep_recursion(int n) {
    if (n == 0) return;
    deep_recursion(n - 1);    // Each call adds a frame
}
```

---

## The Heap

The heap is a region of memory that you control manually. It is used for:

- Dynamically allocated memory (`malloc`, `calloc`, `realloc`)
- Data that needs to outlive the function that created it
- Large data structures

**Characteristics:**

- **Manual allocation:** You must call `malloc`, `calloc`, or `realloc`.
- **Manual deallocation:** You must call `free` when you're done.
- **Flexible size:** The heap can grow as needed (up to available memory).
- **Slower:** Allocation and deallocation require system calls and bookkeeping.
- **Fragmentation:** Repeated allocation and freeing can fragment memory.

**Example:**

```c
int *p = malloc(sizeof(int));    // Allocated on the heap
if (p) {
    *p = 42;
    // ...
    free(p);                     // Must free when done
}
```

**Memory leaks:** If you allocate memory on the heap and never free it, the memory is lost until the program exits.

```c
void leak(void) {
    int *p = malloc(sizeof(int));    // Memory allocated
    // No free — memory leak
}
```

---

## Memory Layout of a C Program

A typical C program's memory is divided into several segments:

```
High addresses
+-------------------+
|       Stack       |  ← Grows downward
|    (local vars)   |
+-------------------+
|                   |
|                   |  ← Free memory
|                   |
+-------------------+
|       Heap        |  ← Grows upward
|  (dynamic alloc)  |
+-------------------+
|  Uninitialized    |
|  data (BSS)       |  ← Global/static variables (zero-initialized)
+-------------------+
|  Initialized      |
|  data             |  ← Global/static variables (initialized)
+-------------------+
|       Text        |  ← Executable code
|    (code)         |
+-------------------+
Low addresses
```

**Segments:**

| Segment | Description |
|---|---|
| **Text** | Executable code (read-only) |
| **Data** | Global/static variables with initial values |
| **BSS** | Global/static variables with zero initial values |
| **Heap** | Dynamically allocated memory (grows upward) |
| **Stack** | Local variables, function frames (grows downward) |

---

## When to Use Stack vs Heap

| Situation | Recommendation |
|---|---|
| Small, fixed-size data | Stack |
| Large data structures | Heap |
| Data that outlives the function | Heap |
| Data of unknown size at compile time | Heap |
| Performance-critical code | Stack (faster) |
| Deep recursion | Beware of stack overflow |
| Variable-length arrays (VLA) | Stack (with caution) |

---

## Common Pitfalls

### 1. Stack Overflow

```c
int large_array[1000000];    // Stack overflow on most systems
```

### 2. Memory Leak

```c
void func(void) {
    int *p = malloc(sizeof(int));
    // Forgot to call free
}
```

### 3. Using Free'd Memory

```c
int *p = malloc(sizeof(int));
free(p);
*p = 10;    // Undefined behavior
```

### 4. Returning a Pointer to a Stack Variable

```c
int *get_value(void) {
    int x = 42;
    return &x;    // Dangerous: x is destroyed
}
```

### 5. Double Free

```c
free(p);
free(p);    // Undefined behavior
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

// Global variable (data segment)
int global = 10;

// Global variable (BSS segment)
static int bss_variable;

void demonstrate_stack(void) {
    // Allocated on the stack
    int local = 42;
    char buffer[100];
    int *p = &local;    // p is on the stack, points to local
    
    printf("Stack variables:\n");
    printf("  local: %d (address: %p)\n", local, (void *)&local);
    printf("  buffer: %p\n", (void *)buffer);
}

void demonstrate_heap(void) {
    // Allocated on the heap
    int *p = malloc(sizeof(int));
    if (p) {
        *p = 42;
        printf("\nHeap variable:\n");
        printf("  *p = %d (address: %p)\n", *p, (void *)p);
        free(p);
    }
}

void demonstrate_lifetime(void) {
    // Stack: destroyed when function returns
    int stack_var = 10;
    
    // Heap: survives function return
    int *heap_var = malloc(sizeof(int));
    if (heap_var) {
        *heap_var = 20;
        // heap_var survives return if we return it or store it
    }
    // But if we don't return it, we lose the pointer
    // heap memory is leaked
}

// Returning a pointer to heap memory (correct)
int *return_heap(void) {
    int *p = malloc(sizeof(int));
    if (p) {
        *p = 42;
    }
    return p;    // Caller must free
}

// Returning a pointer to stack memory (incorrect)
int *return_stack(void) {
    int x = 42;
    return &x;    // Dangerous: x is destroyed
}

int main(void) {
    // Global variables
    printf("Global: %d (address: %p)\n", global, (void *)&global);
    printf("BSS: %d (address: %p)\n", bss_variable, (void *)&bss_variable);
    printf("\n");
    
    demonstrate_stack();
    demonstrate_heap();
    
    // Correct use of heap return
    int *p = return_heap();
    if (p) {
        printf("\nReturned heap pointer: %d\n", *p);
        free(p);
    }
    
    // Memory leak example (bad practice)
    // int *leak = malloc(sizeof(int));    // Leak if not freed
    
    // Stack address (shows the memory layout)
    int stack_var = 10;
    int *heap_var = malloc(sizeof(int));
    if (heap_var) {
        printf("\nAddress comparison:\n");
        printf("  Stack: %p\n", (void *)&stack_var);
        printf("  Heap:  %p\n", (void *)heap_var);
        free(heap_var);
    }
    
    printf("\nsizeof stack: %zu bytes\n", sizeof(stack_var));
    printf("sizeof heap: %zu bytes (pointer)\n", sizeof(heap_var));
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [Project 04 — Pointer Explorer](projects/04-pointer-explorer/)
- **Next:** [32 — Static Allocation](32-static-allocation.md)
- **Dynamic memory:** [33 — Dynamic Allocation](33-dynamic-allocation.md)
- **Memory layout:** [36 — Memory Layout](36-memory-layout.md)

---

## References

- ISO/IEC 9899:2018 §6.2.4 — Storage durations of objects
- ISO/IEC 9899:2018 §7.22 — Memory management functions `<stdlib.h>`