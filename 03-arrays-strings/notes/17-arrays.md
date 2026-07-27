# 17: Arrays — Declaration, Initialization, Access

---

## Overview

An array is a contiguous block of memory containing multiple elements of the same type. It is one of the most fundamental data structures in C. Arrays are the foundation for strings, buffers, and many data structures.

**Key characteristics:**

- All elements are the same type
- Elements are stored contiguously in memory
- The array name is a pointer to the first element
- Array size is fixed at compile time
- No bounds checking (you are responsible)

---

## Declaration

```c
type name[size];
```

**Examples:**

```c
int numbers[10];                // 10 integers
char buffer[256];               // 256 characters
float temperatures[30];         // 30 floats
int matrix[3][4];               // 3x4 2D array
```

**Size must be a constant expression:**

```c
#define SIZE 100
int arr1[SIZE];                 // OK

int size = 50;
int arr2[size];                 // VLA (C99) — see below
```

**Variable-length arrays (VLAs) in C99:**

```c
int n = 10;
int arr[n];                     // VLA: size determined at runtime
```

VLAs are supported in C99 and C11, but are optional in C11 (C17). They are not supported in all environments (especially embedded). Use with caution.

---

## Initialization

### At Declaration

```c
int arr[5] = {1, 2, 3, 4, 5};    // Full initialization
int arr[5] = {1, 2, 3};           // Partial: {1, 2, 3, 0, 0}
int arr[5] = {0};                 // All zeros: {0, 0, 0, 0, 0}
int arr[] = {1, 2, 3};            // Size inferred: 3
```

### Designated Initializers (C99)

```c
int arr[5] = {[2] = 10, [4] = 20};    // {0, 0, 10, 0, 20}
int arr[] = {[2] = 10, [4] = 20};     // Size inferred: 5
```

### Arrays of Strings (Character Arrays)

```c
char str1[] = "Hello";          // Size 6 ('H', 'e', 'l', 'l', 'o', '\0')
char str2[10] = "Hello";        // Size 10, initialized: "Hello\0\0\0\0\0"
char str3[] = {'H', 'e', 'l', 'l', 'o'};    // No null terminator (size 5)
```

**Important:** `"Hello"` is a string literal. It includes a null terminator (`\0`). `{'H', 'e', 'l', 'l', 'o'}` is an array of characters, not a C string (no terminator).

---

## Access

Array elements are accessed by index. Indices start at `0` and go to `size - 1`.

```c
int arr[5] = {10, 20, 30, 40, 50};
int x = arr[0];        // 10
int y = arr[4];        // 50
arr[2] = 100;          // {10, 20, 100, 40, 50}
```

**No bounds checking:**

```c
int arr[5];
arr[5] = 10;    // Out of bounds: undefined behavior
arr[100] = 20;  // Out of bounds: undefined behavior
```

Accessing out of bounds does not cause an error at compile time. It may cause segmentation faults, data corruption, or silent errors.

---

## Arrays and Pointers

The array name is a pointer to the first element:

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;        // p points to arr[0]
*p = 10;            // arr[0] = 10
p[2] = 30;           // arr[2] = 30
```

**The difference between arrays and pointers:**

```c
int arr[5];      // arr is an array, not a pointer
int *p;           // p is a pointer

// sizeof(arr) = 5 * sizeof(int) = 20 (on 32-bit)
// sizeof(p) = sizeof(int *) = 4 (or 8 on 64-bit)
```

```c
arr++;            // Error: arr is not an lvalue
p++;              // OK
```

---

## Passing Arrays to Functions

When passing an array to a function, it decays to a pointer to the first element. You cannot pass an array by value (the entire array is not copied).

```c
void print_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main(void) {
    int arr[] = {1, 2, 3, 4, 5};
    print_array(arr, 5);     // arr decays to pointer
    return 0;
}
```

The function signature `int arr[]` is exactly the same as `int *arr`.

**Modifying the original array:**

```c
void zero_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        arr[i] = 0;          // Modifies the original array
    }
}
```

**`const` modifier:**

```c
void print_array(const int arr[], int size) {
    // arr[i] cannot be modified
}
```

**Static array size (C99):**

```c
void func(int arr[static 10]) {    // arr must have at least 10 elements
    // ...
}
```

---

## 2D Arrays

A 2D array is an array of arrays.

```c
int matrix[3][4];     // 3 rows, 4 columns
```

**Memory layout:** Row-major order. All elements are contiguous in memory: row 0, row 1, row 2.

**Initialization:**

```c
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

int matrix[2][3] = {1, 2, 3, 4, 5, 6};    // Same as above
int matrix[][3] = {1, 2, 3, 4, 5, 6};    // First dimension inferred
```

**Access:**

```c
matrix[0][0] = 10;
matrix[1][2] = 30;
```

**Passing to functions:**

```c
void print_matrix(int rows, int cols, int matrix[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
}
```

Or with fixed dimensions:

```c
void print_matrix(int matrix[][3], int rows) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < 3; j++) {
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
}
```

---

## Common Pitfalls

### 1. Out-of-Bounds Access

```c
int arr[5];
arr[5] = 10;    // UB
arr[-1] = 10;   // UB
```

### 2. Forgetting That Array Name Is a Pointer

```c
int arr[5];
int *p = arr;
int size = sizeof(arr);    // Size of the array
int psize = sizeof(p);     // Size of the pointer
```

### 3. Returning a Pointer to a Local Array

```c
int *get_array(void) {
    int arr[5] = {1, 2, 3, 4, 5};
    return arr;    // Dangerous: arr is destroyed
}
```

### 4. Using `sizeof` on a Parameter

```c
void func(int arr[], int size) {
    int s = sizeof(arr);    // Size of pointer, not array
}
```

### 5. Forgetting the Null Terminator in Strings

```c
char str[5] = "Hello";    // Error: no room for null terminator
char str[] = "Hello";     // OK: size 6
```

---

## Complete Example

```c
#include <stdio.h>

void print_array(const int arr[], int size) {
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

void zero_array(int arr[], int size) {
    for (int i = 0; i < size; i++) {
        arr[i] = 0;
    }
}

void print_matrix(int rows, int cols, int matrix[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%d ", matrix[i][j]);
        }
        printf("\n");
    }
}

void double_matrix(int rows, int cols, int matrix[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            matrix[i][j] *= 2;
        }
    }
}

int main(void) {
    // 1D array
    int arr[5] = {1, 2, 3, 4, 5};
    printf("Original: ");
    print_array(arr, 5);
    
    zero_array(arr, 5);
    printf("Zeroed: ");
    print_array(arr, 5);
    
    // Array pointer relationship
    int *p = arr;
    p[0] = 10;
    p[4] = 50;
    printf("After pointer: ");
    print_array(arr, 5);
    
    // 2D array
    int matrix[2][3] = {
        {1, 2, 3},
        {4, 5, 6}
    };
    printf("\nMatrix:\n");
    print_matrix(2, 3, matrix);
    
    double_matrix(2, 3, matrix);
    printf("Doubled:\n");
    print_matrix(2, 3, matrix);
    
    // String array
    char str1[] = "Hello";
    char str2[10] = "World";
    printf("\nstr1: %s\n", str1);
    printf("str2: %s\n", str2);
    printf("str1 size: %zu\n", sizeof(str1));     // 6
    printf("str2 size: %zu\n", sizeof(str2));     // 10
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [Project 02 — Function Library](projects/02-function-library/)
- **Next:** [18 — Multidimensional Arrays](18-multidimensional.md)
- **Strings:** [19 — Strings](19-strings.md)
- **Pointers:** [23 — Pointer Basics](/04-pointers/notes/23-pointer-basics.md)
- **Array/pointer relationship:** [21 — Array and Pointer Relationship](21-array-pointer.md)

---

## References

- ISO/IEC 9899:2018 §6.7.6.2 — Array declarators
- ISO/IEC 9899:2018 §6.5.2.1 — Array subscripting
- ISO/IEC 9899:2018 §6.7.9 — Initialization