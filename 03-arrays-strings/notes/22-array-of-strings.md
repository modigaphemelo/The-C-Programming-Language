# 22: Array of Strings — `char *argv[]`

---

## Overview

An array of strings is a two-dimensional data structure where each element is a string. In C, this is implemented as an array of pointers to characters (strings). This is commonly used for:

- Command-line arguments (`argv`)
- Lists of names, files, or options
- Small dictionaries or lookup tables

---

## The Two Types of String Arrays

### 1. Array of Pointers to Strings

```c
char *names[] = {"Alice", "Bob", "Charlie"};
```

This is an array of pointers. Each pointer points to a string literal or dynamically allocated string.

### 2. 2D Character Array

```c
char names[3][10] = {"Alice", "Bob", "Charlie"};
```

This is a 2D array. Each row is a fixed-length character array.

---

## Array of Pointers to Strings

This is the most common form. It is flexible, memory-efficient, and widely used.

```c
char *names[] = {"Alice", "Bob", "Charlie"};
```

**Memory layout:**

```
names[0] → "Alice"
names[1] → "Bob"
names[2] → "Charlie"
```

**Access:**

```c
printf("%s\n", names[0]);    // "Alice"
printf("%c\n", names[0][0]); // 'A'
printf("%c\n", names[1][2]); // 'b' (Bob[2] = 'b')
```

**Modifying the strings:**

```c
// Cannot modify string literals
names[0][0] = 'a';    // Undefined behavior (string literal)

// Replace the pointer
names[0] = "Alex";    // OK: now points to "Alex"

// Or use dynamic allocation
char buffer[10] = "Hello";
names[1] = buffer;    // OK: points to modifiable memory
```

---

## 2D Character Array

A 2D character array is a rectangular block of memory where each row has the same length.

```c
char names[3][10] = {"Alice", "Bob", "Charlie"};
```

**Memory layout:**

```
names[0] → [A][l][i][c][e][\0][ ][ ][ ][ ]
names[1] → [B][o][b][\0][ ][ ][ ][ ][ ][ ]
names[2] → [C][h][a][r][l][i][e][\0][ ][ ]
```

**Advantages:**

- All rows are contiguous in memory
- No extra pointer indirection
- Strings are modifiable

**Disadvantages:**

- Fixed size (wastes memory if strings are short, fails if strings are long)
- Less flexible

```c
strcpy(names[0], "Alexander");    // Buffer overflow! (10 bytes is too small)
```

---

## Command-Line Arguments: `argv`

`argv` is an array of strings. It is a classic example of an array of pointers.

```c
int main(int argc, char *argv[]) {
    // argv is an array of strings
    for (int i = 0; i < argc; i++) {
        printf("argv[%d] = %s\n", i, argv[i]);
    }
    return 0;
}
```

**`argc`** — number of arguments (including the program name)
**`argv`** — array of strings (each argument)

```
argv[0] → "./program"
argv[1] → "arg1"
argv[2] → "arg2"
...
argv[argc] → NULL (sentinel)
```

---

## Declaring Arrays of Strings

### 1. As a Local Variable

```c
char *names[] = {"Alice", "Bob", "Charlie"};
```

### 2. As a Function Parameter

```c
void print_names(char *names[], int count) {
    for (int i = 0; i < count; i++) {
        printf("%s\n", names[i]);
    }
}
```

### 3. As a Global

```c
char *names[] = {"Alice", "Bob", "Charlie"};
```

### 4. With Dynamic Allocation

```c
char **names = malloc(3 * sizeof(char *));
names[0] = "Alice";
names[1] = "Bob";
names[2] = "Charlie";
```

### 5. With Dynamic Allocation and Copying

```c
char **names = malloc(3 * sizeof(char *));
for (int i = 0; i < 3; i++) {
    names[i] = malloc(10);          // Allocate space for each string
    strcpy(names[i], "default");
}
```

---

## Reading and Writing Arrays of Strings

### Reading from User Input

```c
char lines[5][100];
int count = 0;

while (count < 5 && fgets(lines[count], sizeof(lines[count]), stdin)) {
    count++;
}

// Remove newline
for (int i = 0; i < count; i++) {
    char *p = strchr(lines[i], '\n');
    if (p) *p = '\0';
}
```

### Sorting an Array of Strings

```c
void sort_strings(char *arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (strcmp(arr[i], arr[j]) > 0) {
                char *temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
}
```

### Copying an Array of Strings

```c
// Copying pointers (shallow copy)
char *copy1[] = {"Alice", "Bob"};

// Duplicating strings (deep copy)
char *copy2[2];
for (int i = 0; i < 2; i++) {
    copy2[i] = malloc(strlen(copy1[i]) + 1);
    strcpy(copy2[i], copy1[i]);
}
```

---

## Common Patterns

### 1. Counting Elements

```c
char *names[] = {"Alice", "Bob", "Charlie"};
int count = sizeof(names) / sizeof(names[0]);    // 3
```

### 2. Searching for a String

```c
int find_string(char *arr[], int count, const char *target) {
    for (int i = 0; i < count; i++) {
        if (strcmp(arr[i], target) == 0) {
            return i;
        }
    }
    return -1;
}
```

### 3. Printing with Indices

```c
void print_with_indices(char *arr[], int count) {
    for (int i = 0; i < count; i++) {
        printf("%d: %s\n", i, arr[i]);
    }
}
```

---

## Common Pitfalls

### 1. Modifying String Literals

```c
char *names[] = {"Alice", "Bob"};
names[0][0] = 'a';    // Undefined behavior
```

**Fix:** Use `const char *names[]` for read-only strings, or use modifiable arrays.

### 2. Buffer Overflow in 2D Character Arrays

```c
char names[2][5] = {"Alice", "Bob"};    // Error: "Alice" needs 6 chars
```

### 3. Forgetting to Allocate Space

```c
char **names = malloc(3 * sizeof(char *));    // Allocates pointers
strcpy(names[0], "Alice");                   // Error: names[0] has no memory
```

### 4. Not Checking Array Bounds

```c
char *names[2] = {"Alice", "Bob"};
printf("%s\n", names[2]);    // Out of bounds: undefined behavior
```

### 5. Forgetting `argc` Check

```c
int main(int argc, char *argv[]) {
    // If no arguments, argv[1] is NULL
    printf("%s\n", argv[1]);    // Undefined behavior if argc < 2
}
```

---

## Complete Example

```c
#include <stdio.h>
#include <string.h>
#include <stdlib.h>

void print_strings(char *arr[], int n) {
    for (int i = 0; i < n; i++) {
        printf("%d: %s\n", i, arr[i]);
    }
}

void sort_strings(char *arr[], int n) {
    for (int i = 0; i < n - 1; i++) {
        for (int j = i + 1; j < n; j++) {
            if (strcmp(arr[i], arr[j]) > 0) {
                char *temp = arr[i];
                arr[i] = arr[j];
                arr[j] = temp;
            }
        }
    }
}

int main(int argc, char *argv[]) {
    printf("Command-line arguments:\n");
    for (int i = 0; i < argc; i++) {
        printf("  argv[%d] = %s\n", i, argv[i]);
    }
    printf("\n");
    
    // Static array of strings
    char *names[] = {"Charlie", "Alice", "Bob"};
    int count = sizeof(names) / sizeof(names[0]);
    
    printf("Original:\n");
    print_strings(names, count);
    
    sort_strings(names, count);
    printf("\nSorted:\n");
    print_strings(names, count);
    
    // 2D character array
    char names2[3][10] = {"Charlie", "Alice", "Bob"};
    printf("\n2D char array:\n");
    for (int i = 0; i < 3; i++) {
        printf("%s\n", names2[i]);
    }
    
    // Strings are modifiable in 2D array
    strcpy(names2[0], "David");
    printf("\nAfter modification:\n");
    for (int i = 0; i < 3; i++) {
        printf("%s\n", names2[i]);
    }
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [21 — Array and Pointer Relationship](21-array-pointer.md)
- **Next:** [Project: String Manipulation Library](projects/03-string-library/)
- **Command-line arguments:** [02 — Program Structure](02-program-structure.md)
- **Dynamic allocation:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)
- **`argv` and `argc`:** [02 — Program Structure](02-program-structure.md)

---

## References

- ISO/IEC 9899:2018 §5.1.2.2 — Program startup (main function)
- ISO/IEC 9899:2018 §6.7.6.2 — Array declarators
- `man argv` — Command-line arguments