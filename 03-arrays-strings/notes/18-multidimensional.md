# 18: Multidimensional Arrays — Rows and Columns

---

## Overview

A multidimensional array is an array of arrays. The most common is a 2D array (matrix), but C supports any number of dimensions. Multidimensional arrays are stored in row-major order: all elements of row 0 first, then row 1, and so on.

**Key characteristics:**

- All elements are the same type
- Stored contiguously in memory (row-major order)
- Multiple dimensions: `int arr[rows][cols]`
- The array name decays to a pointer to the first element (a 1D array)

---

## Declaration and Initialization

### 2D Arrays

```c
type name[rows][cols];
```

**Examples:**

```c
int matrix[3][4];              // 3 rows, 4 columns
float grid[10][10];            // 10x10 grid
char chessboard[8][8];         // Chessboard
```

**Initialization:**

```c
// Full initialization
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

// All in one line (same result)
int matrix[2][3] = {1, 2, 3, 4, 5, 6};

// Partial initialization (remaining elements zero)
int matrix[2][3] = {
    {1, 2},          // {1, 2, 0}
    {4}              // {4, 0, 0}
};

// Zero-initialize
int matrix[2][3] = {0};

// First dimension can be inferred
int matrix[][3] = {
    {1, 2, 3},
    {4, 5, 6}
};
```

### 3D Arrays

```c
int cube[3][4][5];     // 3 layers, 4 rows, 5 columns

// Initialization
int cube[2][2][2] = {
    {
        {1, 2},
        {3, 4}
    },
    {
        {5, 6},
        {7, 8}
    }
};
```

**Designated initializers (C99):**

```c
int matrix[3][3] = {
    [0][0] = 1,
    [1][1] = 2,
    [2][2] = 3
};
// [[1, 0, 0],
//  [0, 2, 0],
//  [0, 0, 3]]
```

---

## Accessing Elements

```c
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

int x = matrix[0][1];    // 2
int y = matrix[1][2];    // 6

matrix[1][0] = 10;       // [[1, 2, 3], [10, 5, 6]]
```

**No bounds checking:**

```c
int matrix[2][3];
matrix[2][0] = 10;    // UB (row index out of bounds)
matrix[0][3] = 10;    // UB (col index out of bounds)
```

---

## Memory Layout

Multidimensional arrays are stored in **row-major order**:

```
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

Memory: [1][2][3][4][5][6]
        ↑ row 0  ↑ row 1
```

**Pointer arithmetic with 2D arrays:**

```c
int matrix[2][3] = {
    {1, 2, 3},
    {4, 5, 6}
};

int *p = (int *)matrix;    // Points to the first element (1)
p[0] = 10;                 // matrix[0][0] = 10
p[3] = 40;                 // matrix[1][0] = 40
```

---

## Passing Multidimensional Arrays to Functions

### With Fixed Dimensions

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

Only the first dimension can be omitted. All other dimensions must be specified.

### With VLA Parameters (C99)

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

### Using Pointers

```c
void print_matrix(int rows, int cols, int *matrix) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%d ", matrix[i * cols + j]);
        }
        printf("\n");
    }
}
```

This is sometimes faster and more flexible but less readable.

### Returning a Multidimensional Array

```c
// Not possible to return a 2D array directly
// Instead, use a pointer to an array:
int (*get_matrix(void))[3] {
    static int matrix[2][3] = {
        {1, 2, 3},
        {4, 5, 6}
    };
    return matrix;
}

int main(void) {
    int (*p)[3] = get_matrix();
    printf("%d\n", p[0][0]);
    return 0;
}
```

---

## Common Patterns

### 1. Matrix Multiplication

```c
void multiply_matrix(int a[][3], int b[][3], int result[][3], int n) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            result[i][j] = 0;
            for (int k = 0; k < n; k++) {
                result[i][j] += a[i][k] * b[k][j];
            }
        }
    }
}
```

### 2. Transpose Matrix

```c
void transpose(int rows, int cols, int src[rows][cols], int dest[cols][rows]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            dest[j][i] = src[i][j];
        }
    }
}
```

### 3. Fill Matrix with Pattern

```c
void identity_matrix(int n, int matrix[n][n]) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            matrix[i][j] = (i == j) ? 1 : 0;
        }
    }
}
```

---

## Common Pitfalls

### 1. Passing an Array of the Wrong Dimensions

```c
void func(int matrix[][4]) { ... }

int m[3][5];    // Wrong: second dimension is 5, not 4
func(m);        // UB
```

### 2. Forgetting the Second Dimension in Pointers

```c
int *p = (int *)matrix;    // Correct
int **p = matrix;          // Wrong: int** is not a 2D array
```

### 3. Using `sizeof` on a Pointer Parameter

```c
void func(int rows, int cols, int matrix[rows][cols]) {
    int s = sizeof(matrix);    // Size of pointer, not array
}
```

### 4. Out-of-Bounds Access

```c
int matrix[2][3];
matrix[2][0] = 10;    // UB
matrix[0][3] = 10;    // UB
```

### 5. Confusing Arrays of Pointers with Multidimensional Arrays

```c
int *arr[3];        // Array of 3 pointers
int arr[3][4];      // 2D array (3 rows, 4 columns)
```

---

## Complete Example

```c
#include <stdio.h>

// VLA parameter (C99)
void print_matrix(int rows, int cols, int matrix[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%4d ", matrix[i][j]);
        }
        printf("\n");
    }
}

// Matrix addition
void add_matrices(int rows, int cols,
                  int a[rows][cols],
                  int b[rows][cols],
                  int result[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            result[i][j] = a[i][j] + b[i][j];
        }
    }
}

// Matrix multiplication (square matrices)
void multiply_matrices(int n, int a[n][n], int b[n][n], int result[n][n]) {
    for (int i = 0; i < n; i++) {
        for (int j = 0; j < n; j++) {
            result[i][j] = 0;
            for (int k = 0; k < n; k++) {
                result[i][j] += a[i][k] * b[k][j];
            }
        }
    }
}

int main(void) {
    // 2x3 matrices
    int a[2][3] = {
        {1, 2, 3},
        {4, 5, 6}
    };
    
    int b[2][3] = {
        {7, 8, 9},
        {10, 11, 12}
    };
    
    int sum[2][3];
    
    printf("Matrix A:\n");
    print_matrix(2, 3, a);
    
    printf("Matrix B:\n");
    print_matrix(2, 3, b);
    
    add_matrices(2, 3, a, b, sum);
    printf("A + B:\n");
    print_matrix(2, 3, sum);
    
    // 3x3 matrices (multiplication)
    int m1[3][3] = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9}
    };
    
    int m2[3][3] = {
        {9, 8, 7},
        {6, 5, 4},
        {3, 2, 1}
    };
    
    int product[3][3];
    
    multiply_matrices(3, m1, m2, product);
    printf("\nMatrix M1:\n");
    print_matrix(3, 3, m1);
    
    printf("Matrix M2:\n");
    print_matrix(3, 3, m2);
    
    printf("M1 * M2:\n");
    print_matrix(3, 3, product);
    
    // 3D array
    int cube[2][2][2] = {
        {
            {1, 2},
            {3, 4}
        },
        {
            {5, 6},
            {7, 8}
        }
    };
    
    printf("\nCube[1][0][1] = %d\n", cube[1][0][1]);    // 6
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [17 — Arrays](17-arrays.md)
- **Next:** [19 — Strings](19-strings.md)
- **Pointer arithmetic:** [25 — Pointer Arithmetic](/04-pointers/notes/25-pointer-arithmetic.md)
- **VLA (C99):** [17 — Arrays](17-arrays.md)

---

## References

- ISO/IEC 9899:2018 §6.7.6.2 — Array declarators
- ISO/IEC 9899:2018 §6.5.2.1 — Array subscripting
- ISO/IEC 9899:2018 §6.7.9 — Initialization