# 44: `typedef` — Creating Type Aliases

---

## Overview

`typedef` is a keyword that creates an alias for an existing type. It does not create a new type—it creates a new name for an existing type. This is useful for:

- Simplifying complex type declarations
- Improving code readability
- Creating portable type names
- Hiding implementation details
- Making code self-documenting

**Key characteristics:**

- Creates a synonym, not a new type
- The compiler treats the new name as the original type
- Improves code readability
- Commonly used with structures, unions, and function pointers
- No runtime overhead

---

## Basic Syntax

```c
typedef existing_type new_name;
```

**Examples:**

```c
typedef int my_int;
typedef char *string;
typedef double (*function_ptr)(double);
```

---

## Common Uses

### 1. Basic Types

```c
typedef int int32_t;
typedef unsigned int uint32_t;
typedef float fp32_t;

int32_t x = 42;    // Same as int x = 42;
uint32_t y = 100;  // Same as unsigned int y = 100;
```

### 2. Structures

```c
// Without typedef
struct Point {
    int x;
    int y;
};
struct Point p;    // Must use 'struct Point'

// With typedef
typedef struct {
    int x;
    int y;
} Point;
Point p;           // No 'struct' keyword needed
```

### 3. Self-Referential Structures

```c
typedef struct Node {
    int value;
    struct Node *next;    // Must use 'struct Node'
} Node;

// Cannot do:
// typedef struct {
//     int value;
//     Node *next;    // Error: Node not defined yet
// } Node;
```

### 4. Unions

```c
typedef union {
    int i;
    float f;
    char *s;
} Variant;

Variant v;    // Instead of 'union Variant v'
```

### 5. Function Pointers

This is where `typedef` really shines. Function pointer syntax is notoriously complex.

```c
// Without typedef
int (*func_ptr)(int, int);

// With typedef
typedef int (*Operation)(int, int);
Operation op = add;

// Array of function pointers
typedef int (*Operation)(int, int);
Operation ops[] = {add, subtract, multiply, divide};
```

### 6. Pointers to Functions Returning Pointers

```c
typedef char *(*StringProcessor)(char *);

char *to_upper(char *s) { /* ... */ }
StringProcessor proc = to_upper;
```

---

## Advantages

### 1. Readability

```c
// Without typedef
struct Date {
    int day;
    int month;
    int year;
};
struct Date today;

// With typedef
typedef struct {
    int day;
    int month;
    int year;
} Date;
Date today;
```

### 2. Portability

```c
// Platform-specific types
#ifdef _WIN32
    typedef unsigned long uint32_t;
#else
    typedef unsigned int uint32_t;
#endif

// Use uint32_t everywhere
uint32_t value = 42;
```

### 3. Self-Documenting Code

```c
typedef float Celsius;
typedef float Fahrenheit;

Celsius temp_c = 25.0;
Fahrenheit temp_f = 77.0;
// The types are the same (both float), but names indicate intent
```

### 4. Simplifying Complex Types

```c
typedef void (*SignalHandler)(int);
typedef char *(*StringArray)[10];

SignalHandler handler = signal_handler;
StringArray names = &name_array;
```

---

## Common Pitfalls

### 1. Hiding Pointers in Typedefs

```c
typedef char *String;

String s = "Hello";
// This is fine, but can be confusing:
String s1, s2;    // Both are char *
```

### 2. Confusing Typedef with Macro

```c
typedef int my_int;    // Creates an alias

#define MY_INT int    // Macro substitution

my_int a = 5;    // int a = 5;
MY_INT b = 5;    // int b = 5; (preprocessor substitution)
```

### 3. Using Typedef for Incomplete Types

```c
typedef struct Node Node;    // Forward declaration
struct Node {
    int value;
    Node *next;    // Can use Node now
};
```

### 4. Hiding Arrays

```c
typedef int Array10[10];
Array10 arr;    // arr is an array of 10 ints
```

### 5. Typedef and Const

```c
typedef char *String;
const String s = "Hello";    // const char *s, not char *const s
// s is a const pointer to char, not a pointer to const char
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Basic typedefs
typedef int Integer;
typedef unsigned int UInteger;
typedef float Real;

// Struct typedef
typedef struct {
    int x;
    int y;
} Point;

// Self-referential struct
typedef struct Node {
    int value;
    struct Node *next;
} Node;

// Union typedef
typedef union {
    int i;
    float f;
    char *s;
} Variant;

// Function pointer typedef
typedef int (*Operation)(int, int);

// Operations
int add(int a, int b) { return a + b; }
int subtract(int a, int b) { return a - b; }
int multiply(int a, int b) { return a * b; }
int divide(int a, int b) { return b != 0 ? a / b : 0; }

// Typedef for complex type
typedef char *(*StringProcessor)(char *);

// String processor functions
char *to_upper(char *s) {
    char *result = strdup(s);
    if (!result) return NULL;
    for (int i = 0; result[i]; i++) {
        if (result[i] >= 'a' && result[i] <= 'z') {
            result[i] -= 32;
        }
    }
    return result;
}

char *to_lower(char *s) {
    char *result = strdup(s);
    if (!result) return NULL;
    for (int i = 0; result[i]; i++) {
        if (result[i] >= 'A' && result[i] <= 'Z') {
            result[i] += 32;
        }
    }
    return result;
}

// Application function using typedef
void apply_operation(int a, int b, Operation op) {
    int result = op(a, b);
    printf("%d\n", result);
}

int main(void) {
    // Basic types
    Integer i = 42;
    UInteger u = 100;
    Real r = 3.14;
    printf("Integer: %d, UInteger: %u, Real: %f\n", i, u, r);
    
    // Struct
    Point p = {10, 20};
    printf("Point: (%d, %d)\n", p.x, p.y);
    
    // Linked list
    Node *head = NULL;
    for (int i = 0; i < 5; i++) {
        Node *node = malloc(sizeof(Node));
        if (!node) continue;
        node->value = i * 10;
        node->next = head;
        head = node;
    }
    
    printf("List: ");
    Node *current = head;
    while (current) {
        printf("%d ", current->value);
        current = current->next;
    }
    printf("\n");
    
    // Free list
    while (head) {
        Node *next = head->next;
        free(head);
        head = next;
    }
    
    // Union
    Variant v;
    v.i = 42;
    printf("Variant int: %d\n", v.i);
    v.f = 3.14;
    printf("Variant float: %f\n", v.f);
    v.s = "Hello";
    printf("Variant string: %s\n", v.s);
    
    // Function pointer typedef
    printf("\nOperations:\n");
    apply_operation(10, 5, add);
    apply_operation(10, 5, subtract);
    apply_operation(10, 5, multiply);
    apply_operation(10, 5, divide);
    
    // Array of function pointers
    Operation ops[] = {add, subtract, multiply, divide};
    const char *names[] = {"add", "subtract", "multiply", "divide"};
    printf("\nArray of operations:\n");
    for (int i = 0; i < 4; i++) {
        printf("%s(10, 5) = %d\n", names[i], ops[i](10, 5));
    }
    
    // String processor
    char original[] = "Hello World!";
    printf("\nOriginal: %s\n", original);
    
    StringProcessor proc = to_upper;
    char *upper = proc(original);
    printf("Upper: %s\n", upper);
    free(upper);
    
    proc = to_lower;
    char *lower = proc(original);
    printf("Lower: %s\n", lower);
    free(lower);
    
    // Typedef for portability
    // (simulating platform-specific types)
    typedef unsigned int uint32_t;
    uint32_t hex_value = 0xDEADBEEF;
    printf("\nuint32_t (hex): 0x%08X\n", hex_value);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [43 — Bit Fields](43-bit-fields.md)
- **Next:** [Project: Data Structure Library](projects/06-data-structures/)
- **Function pointers:** [29 — Function Pointers](29-function-pointers.md)
- **Structures:** [38 — Structures (`struct`)](38-structures.md)
- **Unions:** [42 — Unions](42-unions.md)

---

## References

- ISO/IEC 9899:2018 §6.7.8 — Type definitions
- `man 3 typedef` — Type definitions
- `man 3 stdint` — Fixed-width integer types