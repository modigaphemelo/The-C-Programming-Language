# 25: Pointer Arithmetic — Moving Through Memory

---

## Overview

Pointer arithmetic is the ability to perform arithmetic operations on pointers. It is one of the most powerful—and dangerous—features of C. Pointer arithmetic allows you to traverse arrays, iterate through strings, and manipulate memory directly.

**Key characteristics:**

- Adding an integer to a pointer moves it forward by that many elements
- Subtracting an integer moves it backward
- Pointer arithmetic is scaled by the size of the pointed-to type
- Only meaningful when both pointers point to the same array

---

## The Core Rule

**When you add 1 to a pointer, it moves forward by the size of the type it points to.**

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;          // p points to arr[0]
p++;                   // p now points to arr[1]
```

This is not just adding 1 byte. It adds `sizeof(int)` bytes.

**Visualizing:**

```
Memory: [1][2][3][4][5]
         ^
         p (address 0x1000)
```

After `p++`:

```
Memory: [1][2][3][4][5]
            ^
            p (address 0x1004)
```

---

## Adding and Subtracting Integers

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;          // arr[0]
int *q = p + 2;        // arr[2]
int *r = q - 1;        // arr[1]
```

**`p + n`** moves forward `n` elements.
**`p - n`** moves backward `n` elements.

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr + 2;    // arr[2] = 30
int *q = p - 1;      // arr[1] = 20
```

---

## Subtracting Pointers

Subtracting two pointers gives the number of elements between them.

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = &arr[0];    // address of arr[0]
int *q = &arr[4];    // address of arr[4]
int diff = q - p;    // 4
```

**The result is of type `ptrdiff_t`** (in `<stddef.h>`).

```c
#include <stddef.h>

ptrdiff_t diff = q - p;
printf("%td\n", diff);    // 4
```

**Subtracting pointers only works if both pointers point to the same array.** Otherwise, the result is undefined behavior.

---

## Comparing Pointers

You can compare pointers using relational operators (`<`, `<=`, `>`, `>=`, `==`, `!=`).

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;
int *q = arr + 3;

if (p < q) {
    printf("p points to an earlier element\n");
}
```

**Comparison is only valid if both pointers point to the same array.** Otherwise, the result is undefined behavior.

---

## Pointer Arithmetic with Arrays

### Traversing an Array with a Pointer

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;

for (int i = 0; i < 5; i++) {
    printf("%d ", *p);
    p++;
}
```

### Traversing with `p` and `p < end`

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;
int *end = arr + 5;    // One past the end

while (p < end) {
    printf("%d ", *p);
    p++;
}
```

### Finding the Length of an Array with Pointers

```c
int arr[5] = {10, 20, 30, 40, 50};
int *p = arr;
int *end = arr + 5;
size_t len = end - p;    // 5
```

---

## Pointer Arithmetic with Different Types

```c
int arr_int[5] = {1, 2, 3, 4, 5};
char arr_char[5] = {'a', 'b', 'c', 'd', 'e'};

int *p_int = arr_int;
char *p_char = arr_char;

p_int++;    // Moves forward sizeof(int) bytes (usually 4)
p_char++;   // Moves forward 1 byte
```

The step size depends on the type:

| Type | Size (typical) | Step |
|---|---|---|
| `char *` | 1 byte | +1 |
| `short *` | 2 bytes | +2 |
| `int *` | 4 bytes | +4 |
| `float *` | 4 bytes | +4 |
| `double *` | 8 bytes | +8 |

---

## Common Pitfalls

### 1. Going Out of Bounds

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr + 5;    // One past the end (valid address, but cannot dereference)
*p = 10;             // Undefined behavior

int *q = arr + 6;    // Out of bounds (undefined behavior)
```

### 2. Subtracting Pointers from Different Arrays

```c
int arr1[5] = {1, 2, 3, 4, 5};
int arr2[5] = {6, 7, 8, 9, 10};
int *p = arr1;
int *q = arr2;
int diff = q - p;    // Undefined behavior
```

### 3. Adding to a Pointer in the Wrong Direction

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr + 4;
p += 2;    // Out of bounds
```

### 4. Dereferencing a Pointer One Past the End

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr + 5;    // One past the end
int x = *p;          // Undefined behavior
```

### 5. Confusing Pointer and Array Size

```c
int arr[5] = {1, 2, 3, 4, 5};
int *p = arr;
int size = sizeof(arr);    // 20 (5 * sizeof(int))
int psize = sizeof(p);     // 4 or 8 (sizeof(int *))
```

---

## Complete Example

```c
#include <stdio.h>
#include <stddef.h>

int main(void) {
    int arr[5] = {10, 20, 30, 40, 50};
    int *p = arr;
    
    // Traversing with pointer arithmetic
    printf("Array traversal:\n");
    for (int i = 0; i < 5; i++) {
        printf("arr[%d] = %d, *(p + %d) = %d\n", i, arr[i], i, *(p + i));
    }
    
    // Adding and subtracting
    int *q = p + 2;    // arr[2] = 30
    int *r = q - 1;    // arr[1] = 20
    printf("\nq = p + 2 = %d\n", *q);
    printf("r = q - 1 = %d\n", *r);
    
    // Pointer difference
    int *start = arr;
    int *end = arr + 5;
    ptrdiff_t diff = end - start;
    printf("\nstart = %p, end = %p\n", (void *)start, (void *)end);
    printf("end - start = %td (elements)\n", diff);
    printf("Memory bytes: %td\n", diff * sizeof(int));
    
    // Traversing with start/end
    printf("\nTraversing with start/end:\n");
    int *pos = start;
    while (pos < end) {
        printf("%d ", *pos);
        pos++;
    }
    printf("\n");
    
    // Pointer arithmetic with different types
    char str[] = "Hello";
    char *cp = str;
    char *end_cp = str + 5;    // One past the end
    
    printf("\nString traversal:\n");
    while (cp < end_cp) {
        printf("%c ", *cp);
        cp++;
    }
    printf("\n");
    
    // Step sizes
    printf("\nStep sizes:\n");
    printf("sizeof(char *) step = 1 byte\n");
    printf("sizeof(short *) step = %zu bytes\n", sizeof(short));
    printf("sizeof(int *) step = %zu bytes\n", sizeof(int));
    printf("sizeof(double *) step = %zu bytes\n", sizeof(double));
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [24 — Pointer Operators (`&`, `*`)](24-pointer-operators.md)
- **Next:** [26 — Null Pointers (`NULL`)](26-null.md)
- **Arrays and pointers:** [21 — Array and Pointer Relationship](21-array-pointer.md)
- **Dynamic memory:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)
- **`ptrdiff_t`:** `<stddef.h>`

---

## References

- ISO/IEC 9899:2018 §6.5.6 — Additive operators
- ISO/IEC 9899:2018 §6.7.6.2 — Array declarators
- ISO/IEC 9899:2018 §7.19 — Common definitions `<stddef.h>` (for `ptrdiff_t`)