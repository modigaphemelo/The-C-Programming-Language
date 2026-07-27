# 21: Array and Pointer Relationship — The Deep Connection

---

## Overview

In C, arrays and pointers are intimately connected. In fact, in most contexts, an array name decays to a pointer to its first element. This is not an analogy or a metaphor—it is exactly how the language works.

Understanding this relationship is essential for understanding C. Once you understand it, everything else—string handling, function parameters, dynamic memory—becomes clearer.

---

## The Core Rule

**In most expressions, an array name decays to a pointer to its first element.**

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;        // arr decays to &arr[0]
```

After this assignment, `p` points to `arr[0]`.

```c
arr[2] = 10;          // arr[2] is 10
p[2] = 10;           // Same: p[2] is also 10
*(p + 2) = 10;       // Also the same
```

---

## The Important Distinction

| Aspect | Array | Pointer |
|---|---|---|
| Memory | Allocated at compile time | Points to memory |
| `sizeof` | Size of the entire array | Size of the pointer |
| Assignment | Cannot be assigned | Can be assigned |
| Decay | Decays to pointer | No decay |

```c
int arr[5];
int *p = arr;

sizeof(arr);    // 5 * sizeof(int)
sizeof(p);      // sizeof(int *)
```

```c
arr = p;        // Error: array name is not an lvalue
p = arr;        // OK
```

---

## The Decay in Detail

When an array name appears in an expression, it decays to a pointer to the first element. This happens in almost all contexts except:

1. When used with `sizeof`
2. When used with the `&` (address-of) operator
3. When used as a string literal initializer

```c
int arr[5];

// Decay (most contexts)
int *p = arr;           // arr decays to &arr[0]
func(arr);              // arr decays to pointer

// No decay
size_t s = sizeof(arr); // Size of the entire array
int (*q)[5] = &arr;     // &arr is a pointer to the entire array
```

---

## Array Subscripting and Pointer Arithmetic

Array subscripting `arr[i]` is exactly equivalent to `*(arr + i)`.

```c
int arr[5] = {10, 20, 30, 40, 50};

int x = arr[2];        // 30
int y = *(arr + 2);    // 30

arr[2] = 100;          // arr[2] = 100
*(arr + 2) = 100;      // Same
```

**Interesting side effect:** `i[arr]` is also valid syntax because `i[arr]` is `*(i + arr)`.

```c
int x = 2[arr];        // 30 (legal but confusing, avoid)
```

---

## Passing Arrays to Functions

When you pass an array to a function, it decays to a pointer.

```c
void print_array(int arr[], int size) {
    // arr is actually a pointer
    // sizeof(arr) is sizeof(int *)
}
```

**These three function declarations are identical:**

```c
void func(int arr[]);
void func(int arr[10]);
void func(int *arr);
```

---

## Strings as Arrays of Characters

Strings are arrays of characters terminated by `\0`. They follow the same pointer decay rules.

```c
char str[] = "Hello";
char *p = str;        // p points to 'H'

printf("%c\n", *p);   // 'H'
printf("%c\n", p[1]); // 'e'
```

**String literals are also arrays:**

```c
char *s = "Hello";    // s points to the first character of the string literal
const char *s = "Hello"; // Better: const indicates read-only
```

---

## 2D Arrays and Pointers

A 2D array is an array of arrays. It decays to a pointer to the first row.

```c
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

// matrix decays to pointer to the first row
int (*p)[3] = matrix;    // p points to the first row (an array of 3 ints)

p[0][0] = 10;            // matrix[0][0] = 10
p[1][1] = 50;            // matrix[1][1] = 50
```

**Pointer to a 2D array:**

```c
int (*p)[3] = matrix;    // Pointer to an array of 3 ints
```

**This is different from:**

```c
int *p = (int *)matrix;  // Pointer to the first element
```

**The difference:**

```c
int (*p)[3] = matrix;    // p + 1 moves to the next row
int *q = (int *)matrix;  // q + 1 moves to the next element
```

---

## Arrays of Pointers

An array of pointers is different from a 2D array.

```c
// Array of pointers
char *names[3] = {"Alice", "Bob", "Charlie"};

// Each element is a pointer to a string literal
// names[0] -> "Alice"
// names[1] -> "Bob"
// names[2] -> "Charlie"
```

**Access:**

```c
printf("%s\n", names[0]);    // "Alice"
printf("%c\n", names[0][0]); // 'A'
```

---

## Summary Table

| Expression | Type | Notes |
|---|---|---|
| `arr` (array name) | Decays to `T *` | Points to first element |
| `&arr` | `T (*)[N]` | Pointer to entire array |
| `arr[i]` | `T` | Equivalent to `*(arr + i)` |
| `&arr[i]` | `T *` | Pointer to element i |
| `arr + i` | `T *` | Pointer to element i |
| `*(arr + i)` | `T` | Equivalent to `arr[i]` |
| `i[arr]` | `T` | Equivalent to `arr[i]` (avoid) |

---

## Common Pitfalls

### 1. Treating Arrays as Pointers

```c
int arr[5];
int *p = arr;        // OK
arr = p;             // Error: array name is not an lvalue
```

### 2. Confusing `sizeof` for Array and Pointer

```c
int arr[10];
int *p = arr;

sizeof(arr);    // 40 (on 32-bit)
sizeof(p);      // 4 (on 32-bit)
```

### 3. Using `sizeof` on a Pointer Parameter

```c
void func(int arr[]) {
    int size = sizeof(arr);    // Size of pointer, not array
}
```

### 4. Treating a 2D Array as a Pointer to Pointer

```c
int matrix[2][3];
int **p = matrix;    // Wrong: matrix decays to int (*)[3], not int **
```

### 5. Out-of-Bounds Access

```c
int arr[5];
int *p = arr;
p[5] = 10;    // UB
```

---

## Complete Example

```c
#include <stdio.h>

void print_array(int arr[], int size) {
    printf("arr[] parameter: sizeof(arr) = %zu\n", sizeof(arr));
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

void print_ptr(int *arr, int size) {
    printf("int * parameter: sizeof(arr) = %zu\n", sizeof(arr));
    for (int i = 0; i < size; i++) {
        printf("%d ", arr[i]);
    }
    printf("\n");
}

int main(void) {
    // 1D array
    int arr[5] = {1, 2, 3, 4, 5};
    int *p = arr;
    
    printf("sizeof(arr) = %zu\n", sizeof(arr));    // 20
    printf("sizeof(p) = %zu\n", sizeof(p));        // 4 or 8
    
    // Pointer arithmetic
    printf("\nPointer arithmetic:\n");
    for (int i = 0; i < 5; i++) {
        printf("arr[%d] = %d, *(p + %d) = %d\n", i, arr[i], i, *(p + i));
    }
    
    // Function calls (both are equivalent)
    printf("\nprint_array:\n");
    print_array(arr, 5);
    
    printf("\nprint_ptr:\n");
    print_ptr(arr, 5);
    
    // 2D array
    int matrix[2][3] = {
        {1, 2, 3},
        {4, 5, 6}
    };
    
    int (*row)[3] = matrix;    // Pointer to first row
    printf("\nmatrix[1][1] = %d\n", matrix[1][1]);
    printf("row[1][1] = %d\n", row[1][1]);
    
    // Arrays of pointers
    char *names[3] = {"Alice", "Bob", "Charlie"};
    printf("\nNames:\n");
    for (int i = 0; i < 3; i++) {
        printf("%s\n", names[i]);
    }
    
    // String as array of characters
    char str[] = "Hello";
    char *s = str;
    printf("\nstr: %s\n", str);
    printf("s: %s\n", s);
    printf("str[0] = %c, *s = %c\n", str[0], *s);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [20 — String Functions](20-string-functions.md)
- **Next:** [22 — Array of Strings](22-array-of-strings.md)
- **Pointer basics:** [23 — Pointer Basics](/04-pointers/notes/23-pointer-basics.md)
- **Pointer arithmetic:** [25 — Pointer Arithmetic](/04-pointers/notes/25-pointer-arithmetic.md)

---

## References

- ISO/IEC 9899:2018 §6.3.2.1 — Lvalues, arrays, and function designators
- ISO/IEC 9899:2018 §6.5.2.1 — Array subscripting
- ISO/IEC 9899:2018 §6.5.6 — Additive operators