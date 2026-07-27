# 29: Function Pointers — Callbacks, Jump Tables

---

## Overview

A function pointer is a variable that stores the address of a function. This allows you to:

- Pass functions as arguments to other functions (callbacks)
- Store functions in arrays (jump tables)
- Call functions dynamically at runtime
- Implement state machines and event handlers

Function pointers are one of the most powerful features of C. They enable polymorphism, callbacks, and dynamic dispatch.

---

## Declaration Syntax

The syntax for declaring function pointers is notoriously tricky. The rule is: **the function pointer must match the function's signature exactly.**

```c
return_type (*pointer_name)(parameter_types);
```

**Examples:**

```c
// Pointer to a function taking two ints and returning int
int (*func)(int, int);

// Pointer to a function taking void and returning void
void (*func)(void);

// Pointer to a function taking a char* and returning a char*
char *(*func)(char *);

// Pointer to a function returning a pointer to int
int *(*func)(int);
```

**Reading declarations:**

Read from right to left:

```c
int (*p)(int, int);    // p is a pointer to a function taking int,int returning int
void (*q)(void);       // q is a pointer to a function taking void returning void
```

---

## Assigning to Function Pointers

```c
int add(int a, int b) {
    return a + b;
}

int main(void) {
    int (*p)(int, int) = add;    // p points to add
    int result = p(5, 3);        // Calls add(5, 3)
    printf("%d\n", result);      // 8
    return 0;
}
```

**You can also use `&`:**

```c
int (*p)(int, int) = &add;    // Works, but unnecessary
```

**Both forms are equivalent.** The function name decays to a pointer to the function.

---

## Calling Functions Through Function Pointers

```c
int add(int a, int b) {
    return a + b;
}

int main(void) {
    int (*p)(int, int) = add;
    int result = p(5, 3);      // Call through pointer
    // Or:
    result = (*p)(5, 3);       // Also works (dereference first)
    return 0;
}
```

Both `p(5, 3)` and `(*p)(5, 3)` work. Most programmers use `p(5, 3)`.

---

## Function Pointers as Parameters (Callbacks)

This is the most common use case. Passing a function pointer to another function allows the other function to call your function when needed.

```c
#include <stdio.h>

// Callback function type
typedef int (*operation_t)(int, int);

int add(int a, int b) {
    return a + b;
}

int multiply(int a, int b) {
    return a * b;
}

int apply_operation(int a, int b, operation_t op) {
    return op(a, b);
}

int main(void) {
    printf("add: %d\n", apply_operation(5, 3, add));       // 8
    printf("multiply: %d\n", apply_operation(5, 3, multiply)); // 15
    return 0;
}
```

---

## The `typedef` Approach

The syntax for function pointers can be messy. A `typedef` makes it cleaner.

```c
// Define a type for a function pointer
typedef int (*operation_t)(int, int);

int add(int a, int b) {
    return a + b;
}

int main(void) {
    operation_t op = add;    // op is now a function pointer
    int result = op(5, 3);   // 8
    return 0;
}
```

**`typedef` syntax for function pointers:**

```c
typedef return_type (*name)(parameter_types);
```

---

## Arrays of Function Pointers (Jump Tables)

Function pointers can be stored in arrays. This creates a jump table, which is useful for:

- State machines
- Command interpreters
- Dispatch tables

```c
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
int multiply(int a, int b) { return a * b; }
int divide(int a, int b) { return (b != 0) ? a / b : 0; }

int main(void) {
    // Array of function pointers
    int (*ops[4])(int, int) = {add, subtract, multiply, divide};
    char *names[] = {"add", "subtract", "multiply", "divide"};
    
    int a = 10, b = 5;
    for (int i = 0; i < 4; i++) {
        printf("%s: %d\n", names[i], ops[i](a, b));
    }
    return 0;
}
```

---

## Sorting with Function Pointers (`qsort`)

The standard library's `qsort` function uses a function pointer to compare elements.

```c
#include <stdlib.h>
#include <stdio.h>

int compare_int(const void *a, const void *b) {
    int x = *(const int *)a;
    int y = *(const int *)b;
    return (x > y) - (x < y);    // -1, 0, or 1
}

int main(void) {
    int arr[] = {5, 2, 8, 1, 9, 3};
    size_t n = sizeof(arr) / sizeof(arr[0]);
    
    qsort(arr, n, sizeof(int), compare_int);
    
    for (size_t i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
    return 0;
}
```

---

## Member Function Pointers (in Structs)

Function pointers can be stored in structures, creating objects with methods.

```c
#include <stdio.h>

typedef struct {
    int x;
    int y;
    int (*add)(struct Point *p);
} Point;

int point_add(Point *p) {
    return p->x + p->y;
}

int main(void) {
    Point p = {10, 20, point_add};
    printf("sum: %d\n", p.add(&p));    // 30
    return 0;
}
```

---

## Common Pitfalls

### 1. Incorrect Signature

```c
int add(int a, int b);
void *(*p)(int, int) = add;    // Warning: incompatible pointer types
```

### 2. Forgetting the `*` in Declaration

```c
int p(int, int);    // Function declaration, not function pointer
int (*p)(int, int); // Function pointer
```

### 3. Calling Without Initialization

```c
int (*p)(int, int);
int result = p(5, 3);    // Undefined behavior: p is uninitialized
```

### 4. Casting Function Pointers

```c
int add(int a, int b);
void (*p)(int, int) = (void (*)(int, int))add;    // Dangerous
```

### 5. Returning a Function Pointer

```c
int (*get_op(void))(int, int) {
    return add;
}
```

**Better with `typedef`:**

```c
typedef int (*op_t)(int, int);
op_t get_op(void) {
    return add;
}
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Function pointer type
typedef int (*op_t)(int, int);

// Operations
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
int multiply(int a, int b) { return a * b; }
int divide(int a, int b) { return (b != 0) ? a / b : 0; }

// Callback
int apply(int a, int b, op_t op) {
    return op(a, b);
}

// Compare function for qsort
int compare_strings(const void *a, const void *b) {
    const char *s1 = *(const char **)a;
    const char *s2 = *(const char **)b;
    return strcmp(s1, s2);
}

// Function returning a function pointer
op_t get_operation(const char *name) {
    if (strcmp(name, "add") == 0) return add;
    if (strcmp(name, "subtract") == 0) return subtract;
    if (strcmp(name, "multiply") == 0) return multiply;
    if (strcmp(name, "divide") == 0) return divide;
    return NULL;
}

int main(void) {
    // Basic function pointer
    int (*p)(int, int) = add;
    printf("add: %d\n", p(5, 3));
    
    // Callback
    printf("multiply (callback): %d\n", apply(5, 3, multiply));
    
    // Jump table
    op_t ops[] = {add, subtract, multiply, divide};
    char *names[] = {"add", "subtract", "multiply", "divide"};
    int a = 10, b = 5;
    
    printf("\nJump table:\n");
    for (int i = 0; i < 4; i++) {
        printf("%s: %d\n", names[i], ops[i](a, b));
    }
    
    // qsort with function pointer
    char *words[] = {"banana", "apple", "cherry", "date"};
    size_t n = sizeof(words) / sizeof(words[0]);
    qsort(words, n, sizeof(char *), compare_strings);
    
    printf("\nSorted words:\n");
    for (size_t i = 0; i < n; i++) {
        printf("%s\n", words[i]);
    }
    
    // Function returning function pointer
    printf("\nDynamic dispatch:\n");
    op_t op = get_operation("multiply");
    if (op) {
        printf("multiply: %d\n", op(6, 7));
    }
    
    // Function pointer in struct
    typedef struct {
        int x;
        int y;
        int (*add)(int, int);
    } Point;
    
    Point pt = {10, 20, add};
    printf("\nStruct method: %d\n", pt.add(pt.x, pt.y));
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [28 — Pointers to Pointers (`**`)](28-pointer-to-pointer.md)
- **Next:** [30 — Void Pointers (`void*`)](30-void-pointers.md)
- **`qsort`:** `man qsort`
- **`typedef`:** [44 — `typedef`](/06-structures-unions/notes/44-typedef.md)
- **State machines:** [75 — Threads](/11-system/notes/75-threads.md)

---

## References

- ISO/IEC 9899:2018 §6.7.6.3 — Function declarators
- ISO/IEC 9899:2018 §6.5.2.2 — Function calls
- ISO/IEC 9899:2018 §6.7.8 — Type definitions