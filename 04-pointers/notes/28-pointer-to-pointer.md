# 28: Pointers to Pointers — `**`

---

## Overview

A pointer to a pointer is exactly what it sounds like: a pointer that stores the address of another pointer. It adds another level of indirection. This is used for:

- Dynamic 2D arrays (arrays of pointers)
- Modifying pointer arguments in functions
- Linked lists (head pointers)
- Tree and graph structures
- Command-line argument arrays (`char **argv`)

---

## Declaration and Assignment

```c
int x = 42;
int *p = &x;     // p points to x
int **pp = &p;   // pp points to p
```

**Visualizing:**

```
x:  [42]       at address 0x1000
p:  [0x1000]   at address 0x2000 (points to x)
pp: [0x2000]   at address 0x3000 (points to p)
```

**Dereferencing:**

```c
int x = 42;
int *p = &x;
int **pp = &p;

printf("%d\n", *p);     // 42
printf("%d\n", **pp);   // 42
```

The double dereference `**pp`:

1. `*pp` → gets the value of `p` (the address of `x`)
2. `**pp` → gets the value at that address (42)

---

## Why Use Pointers to Pointers?

### 1. Modifying a Pointer Argument

When you pass a pointer to a function, the function receives a copy of the pointer. To modify the pointer itself (not the thing it points to), you need a pointer to the pointer.

```c
void allocate_bad(char *buffer) {
    buffer = malloc(100);    // Only modifies local copy
}

void allocate_good(char **buffer) {
    *buffer = malloc(100);   // Modifies the original pointer
}

int main(void) {
    char *buf = NULL;
    allocate_bad(buf);       // buf remains NULL
    allocate_good(&buf);     // buf now points to allocated memory
    free(buf);
    return 0;
}
```

### 2. Dynamic 2D Arrays

A 2D array can be implemented as an array of pointers, each pointing to a row.

```c
int rows = 3, cols = 4;
int **matrix = malloc(rows * sizeof(int *));
for (int i = 0; i < rows; i++) {
    matrix[i] = malloc(cols * sizeof(int));
}

// Access: matrix[row][col]
matrix[0][0] = 10;
matrix[1][2] = 20;

// Free
for (int i = 0; i < rows; i++) {
    free(matrix[i]);
}
free(matrix);
```

### 3. Linked Lists

A pointer to a pointer is often used to traverse and modify linked lists.

```c
struct Node {
    int value;
    struct Node *next;
};

void push(struct Node **head, int value) {
    struct Node *new_node = malloc(sizeof(struct Node));
    new_node->value = value;
    new_node->next = *head;
    *head = new_node;    // Modifies the head pointer
}

int main(void) {
    struct Node *head = NULL;
    push(&head, 10);
    push(&head, 20);
    // head now points to the first node
    return 0;
}
```

### 4. `argv` (Command-Line Arguments)

```c
int main(int argc, char **argv) {
    // argv is a pointer to an array of strings
    // argv[0] is the program name
    // argv[argc] is NULL
    return 0;
}
```

`char **argv` is equivalent to `char *argv[]`. It's a pointer to the first string in the array.

---

## Common Pitfalls

### 1. Dereferencing the Wrong Level

```c
int x = 42;
int *p = &x;
int **pp = &p;

printf("%d\n", *pp);    // Prints address of x (not 42)
printf("%d\n", **pp);   // Prints 42 (correct)
```

### 2. Forgetting the `&` When Passing

```c
void func(int **p) { ... }

int main(void) {
    int *p = NULL;
    func(p);     // Error: passing int * where int ** is expected
    func(&p);    // Correct
    return 0;
}
```

### 3. Uninitialized Pointer to Pointer

```c
int **pp;
**pp = 42;    // Undefined behavior (pp points to nothing)
```

### 4. Confusing Arrays and Pointers to Pointers

```c
int matrix[3][4];    // 2D array (contiguous memory)
int **p = matrix;    // Wrong: matrix is int (*)[4], not int **
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

void modify_pointer(int **pp) {
    static int static_value = 99;
    *pp = &static_value;    // Make pp point to static_value
}

void allocate_matrix(int ***matrix, int rows, int cols) {
    *matrix = malloc(rows * sizeof(int *));
    for (int i = 0; i < rows; i++) {
        (*matrix)[i] = malloc(cols * sizeof(int));
    }
}

void free_matrix(int **matrix, int rows) {
    for (int i = 0; i < rows; i++) {
        free(matrix[i]);
    }
    free(matrix);
}

void print_matrix(int **matrix, int rows, int cols) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
}

int main(void) {
    // Basic pointer to pointer
    int x = 42;
    int *p = &x;
    int **pp = &p;
    
    printf("x = %d\n", x);
    printf("*p = %d\n", *p);
    printf("**pp = %d\n", **pp);
    printf("pp points to p at address %p\n", (void *)pp);
    printf("p points to x at address %p\n", (void *)p);
    
    // Modifying pointer argument
    int *ptr = NULL;
    modify_pointer(&ptr);
    if (ptr != NULL) {
        printf("\nAfter modify_pointer: *ptr = %d\n", *ptr);
    }
    
    // Dynamic 2D array
    int **matrix;
    int rows = 3, cols = 4;
    
    allocate_matrix(&matrix, rows, cols);
    
    // Fill with values
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            matrix[i][j] = i * cols + j;
        }
    }
    
    printf("\nMatrix:\n");
    print_matrix(matrix, rows, cols);
    
    free_matrix(matrix, rows);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [27 — Pointer Parameters](27-pointer-parameters.md)
- **Next:** [29 — Function Pointers](29-function-pointers.md)
- **Void pointers:** [30 — Void Pointers](30-void-pointers.md)
- **Dynamic 2D arrays:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)

---

## References

- ISO/IEC 9899:2018 §6.3.2.3 — Pointers
- ISO/IEC 9899:2018 §6.5.3.2 — Address and indirection operators