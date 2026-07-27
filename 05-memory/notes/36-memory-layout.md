# 36: Memory Layout — Text, Data, BSS, Heap, Stack

---

## Overview

When a C program runs, its memory is divided into several distinct segments. Each segment serves a different purpose and has different characteristics. Understanding this layout helps you understand where variables live, what happens when you allocate memory, and why certain errors occur.

**The five primary segments:**

| Segment | Contents | Read/Write |
|---|---|---|
| **Text** | Executable code | Read-only |
| **Data** | Initialized global/static variables | Read-write |
| **BSS** | Uninitialized global/static variables | Read-write (zero-initialized) |
| **Heap** | Dynamically allocated memory | Read-write (grows upward) |
| **Stack** | Local variables, function frames | Read-write (grows downward) |

---

## Full Memory Layout

```
High addresses
+------------------+
|      Stack       |  ← Grows downward (toward lower addresses)
|    (local vars)  |
|                  |
|  +------------+  |
|  |   Stack    |  |  ← Each function call adds a frame
|  |   Frame    |  |
|  +------------+  |
|  |   Stack    |  |
|  |   Frame    |  |
|  +------------+  |
+------------------+
|                  |
|  (free memory)   |
|                  |
+------------------+
|      Heap        |  ← Grows upward (toward higher addresses)
|  (dynamic alloc) |
|                  |
|  +------------+  |
|  |  malloc()  |  |
|  +------------+  |
|  |  malloc()  |  |
|  +------------+  |
+------------------+
|       BSS        |  ← Uninitialized global/static data (zero-initialized)
|  (uninitialized) |
+------------------+
|      Data        |  ← Initialized global/static data
|  (initialized)   |
+------------------+
|      Text        |  ← Executable code (read-only)
|    (code)        |
+------------------+
Low addresses
```

---

## The Text Segment

The text segment contains the executable machine code of the program. It is read-only to prevent accidental modification.

**Characteristics:**

- Contains compiled code
- Read-only (cannot be modified)
- Shared between processes (if running multiple copies)
- Size is fixed at compile time

```c
int main(void) {
    // main() code lives in the text segment
    return 0;
}
```

**String literals:**
String literals (like `"Hello"`) are typically stored in the text segment (or a separate read-only data section). This is why modifying them is undefined behavior.

```c
char *s = "Hello";    // "Hello" is in read-only memory
s[0] = 'h';           // Segmentation fault (read-only memory)
```

---

## The Data Segment

The data segment contains initialized global and static variables. It is read-write and exists for the entire program lifetime.

**Contents:**

- Global variables with initial values
- Static variables with initial values
- Constants (sometimes)

```c
int global = 10;              // Data segment
static int file_static = 20;  // Data segment

void func(void) {
    static int local_static = 30;    // Data segment
}
```

**Read-only data:**
Some constants (like `const` globals) may be placed in a read-only data section (`.rodata`).

```c
const int readonly = 42;    // May be in .rodata (read-only)
```

---

## The BSS Segment

BSS stands for "Block Started by Symbol" (historical). It contains uninitialized global and static variables. The OS zero-initializes the entire BSS segment at program startup.

**Contents:**

- Global variables without an initializer
- Static variables without an initializer
- Variables explicitly initialized to zero

```c
int global_zero;              // BSS segment (0)
static int static_zero;       // BSS segment (0)
static int zero_init = 0;     // BSS segment (0)

void func(void) {
    static int local_static;  // BSS segment (0)
}
```

**Why BSS matters:**

- Saves space in the executable file
- The OS zeroes it quickly at startup
- Large arrays don't make the executable larger

```c
// .bss: allocated at runtime, not in the executable file
static char huge_buffer[1000000];    // 1MB, but doesn't increase file size
```

---

## The Heap

The heap is the region of memory used for dynamic allocation (`malloc`, `calloc`, `realloc`). It grows upward (toward higher addresses) as more memory is allocated.

**Characteristics:**

- Managed manually (you call `malloc`/`free`)
- Grows upward
- Memory can be allocated at any time
- Fragmentation is possible

```c
int *p = malloc(sizeof(int));    // Allocated on the heap
char *s = malloc(100);           // Allocated on the heap
```

**Heap growth:**

When you allocate memory, the heap manager asks the OS for more memory (using `brk`/`sbrk` or `mmap`). The heap grows upward toward the stack.

```
Stack grows down ←
        |
        |
        |  Free memory
        |
        |
Heap grows up   ↑
```

---

## The Stack

The stack is used for function call frames and local variables. It grows downward (toward lower addresses).

**Contents of a stack frame:**

- Function parameters
- Local variables
- Return address
- Saved registers

```c
void func(int param) {
    int local = 42;    // Allocated on the stack
    char buffer[100];  // Allocated on the stack
}
```

**Stack growth:**

Each function call pushes a new frame onto the stack. When the function returns, the frame is popped.

```
Stack pointer (SP) →   [Current frame]
                       [Previous frame]
                       [Older frame]
```

**Stack overflow:**

If you allocate too much on the stack (large arrays, deep recursion), the stack can grow into the heap, causing a stack overflow.

```c
void recursion(int n) {
    int arr[1000];        // Each call allocates 4KB on the stack
    if (n == 0) return;
    recursion(n - 1);     // Deep recursion can overflow
}
```

---

## Address Space Layout Randomization (ASLR)

Modern operating systems randomize the base addresses of memory segments to prevent security attacks. This is why addresses change between runs.

```c
printf("%p\n", (void *)main);    // Text address changes each run
printf("%p\n", (void *)&global); // Data address changes each run
```

---

## Visualizing Memory Layout

```c
#include <stdio.h>
#include <stdlib.h>

// Text: code
int main(void) {
    // Data: initialized global
    int global = 10;
    
    // BSS: zero-initialized global
    static int bss_var;
    
    // Stack: local variables
    int stack_var = 20;
    
    // Heap: dynamic allocation
    int *heap_var = malloc(sizeof(int));
    
    printf("Text:   main = %p\n", (void *)main);
    printf("Data:   global = %p\n", (void *)&global);
    printf("BSS:    bss_var = %p\n", (void *)&bss_var);
    printf("Heap:   heap_var = %p\n", (void *)heap_var);
    printf("Stack:  stack_var = %p\n", (void *)&stack_var);
    
    free(heap_var);
    return 0;
}
```

**Typical output:**

```
Text:   main = 0x401000
Data:   global = 0x404000
BSS:    bss_var = 0x405000
Heap:   heap_var = 0x7fff00000000
Stack:  stack_var = 0x7fffffffe000
```

---

## Memory Segment Sizes

| Segment | Typical Size | Growth |
|---|---|---|
| Text | Fixed (executable size) | None |
| Data | Fixed (global size) | None |
| BSS | Fixed (global size) | None |
| Heap | Dynamic (up to available memory) | Upward |
| Stack | Fixed (1-8 MB typical) | Downward |

**Stack size limits:**

```bash
# Linux
ulimit -s        # Show stack size (KB)
ulimit -s 8192   # Set to 8MB
```

**Viewing memory maps (Linux):**

```bash
cat /proc/self/maps        # Memory map of the cat command
pmap <pid>                 # Memory map of a running process
```

---

## Common Pitfalls

### 1. Stack Overflow

```c
int large_array[1000000];    // Stack overflow
```

**Fix:** Use `static` or `malloc` for large arrays.

### 2. Heap/Stack Collision

If the heap grows too far upward or the stack grows too far downward, they can collide. This usually causes a crash.

### 3. Accessing Freed Memory

```c
free(p);
*p = 10;    // Undefined behavior (may corrupt heap metadata)
```

### 4. Writing to the Text Segment

```c
char *s = "Hello";
s[0] = 'h';    // Segmentation fault (read-only memory)
```

### 5. Assuming a Fixed Layout

Memory layout is randomized (ASLR). Don't assume fixed addresses.

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

// Data segment (initialized)
int global = 10;

// BSS segment (zero-initialized)
static int bss_var;

// Text segment
void function(void) {
    // Stack (local variables)
    int local = 42;
    printf("function: local = %d\n", local);
}

int main(void) {
    // Stack
    int stack_var = 20;
    int *heap_var = malloc(sizeof(int));
    
    if (heap_var == NULL) {
        return 1;
    }
    *heap_var = 100;
    
    // Static variable (Data segment)
    static int static_var = 30;
    
    // Uninitialized static (BSS segment)
    static int static_uninit;
    
    printf("=== Memory Layout ===\n");
    printf("Text (code):       main = %p\n", (void *)main);
    printf("Text (code):       function = %p\n", (void *)function);
    printf("Data (initialized): global = %p (value: %d)\n", (void *)&global, global);
    printf("Data (initialized): static_var = %p (value: %d)\n", (void *)&static_var, static_var);
    printf("BSS (zero):        bss_var = %p (value: %d)\n", (void *)&bss_var, bss_var);
    printf("BSS (zero):        static_uninit = %p (value: %d)\n", (void *)&static_uninit, static_uninit);
    printf("Heap:              heap_var = %p (value: %d)\n", (void *)heap_var, *heap_var);
    printf("Stack:             stack_var = %p (value: %d)\n", (void *)&stack_var, stack_var);
    
    // Show stack frames
    function();
    
    // Show that heap is between stack and data
    printf("\n=== Address Order ===\n");
    printf("Stack (highest):   %p\n", (void *)&stack_var);
    printf("Heap:              %p\n", (void *)heap_var);
    printf("BSS:               %p\n", (void *)&bss_var);
    printf("Data:              %p\n", (void *)&global);
    printf("Text (lowest):     %p\n", (void *)main);
    
    free(heap_var);
    return 0;
}
```

---

## Cross-References

- **Previous:** [35 — Memory Leaks](35-memory-leaks.md)
- **Next:** [37 — Alignment and Padding](37-alignment.md)
- **Stack vs heap:** [31 — Stack vs Heap](31-stack-heap.md)
- **Static allocation:** [32 — Static Allocation](32-static-allocation.md)
- **Dynamic allocation:** [33 — Dynamic Allocation](33-dynamic-allocation.md)
- **Stack overflow:** [84 — Undefined Behavior](/13-ub/notes/84-ub.md)
- **ASLR:** Security hardening

---

## References

- ISO/IEC 9899:2018 §6.2.4 — Storage durations
- `man 5 elf` — ELF file format
- `man 3 brk` — Heap management
- `ulimit -a` — System limits