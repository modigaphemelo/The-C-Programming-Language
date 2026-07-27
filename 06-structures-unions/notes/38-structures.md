# 38: Structures — `struct`, Grouping Data

---

## Overview

A structure is a user-defined data type that groups related variables together under a single name. Each variable in the structure is called a member. Structures are the foundation of data organization in C—they are used to represent records, objects, and complex data.

**Key characteristics:**

- Groups multiple variables of different types
- Members are stored contiguously in memory (with padding)
- Access members using dot (`.`) or arrow (`->`) operators
- Can contain any type: primitives, arrays, pointers, other structs
- Can be passed to functions (by value or by pointer)
- Can be returned from functions

---

## Declaring a Structure

### Basic Declaration

```c
struct Point {
    int x;
    int y;
};
```

This defines a new type: `struct Point`. It has two members: `x` and `y`.

### Declaring Variables

```c
struct Point p1;              // p1 is a variable of type struct Point
struct Point p2 = {10, 20};   // Initialization
```

### Typedef

Using `typedef` makes the type name cleaner:

```c
typedef struct {
    int x;
    int y;
} Point;

Point p1;    // No "struct" keyword needed
```

### Multiple Variables

```c
struct Point p1, p2, p3;
```

---

## Accessing Members

Use the dot operator (`.`) to access members of a structure variable.

```c
struct Point p = {10, 20};
p.x = 30;
p.y = 40;

printf("(%d, %d)\n", p.x, p.y);    // (30, 40)
```

### Arrays of Structures

```c
struct Point points[5];
points[0].x = 10;
points[0].y = 20;
```

---

## Initialization

### Aggregated Initialization

```c
struct Point p1 = {10, 20};            // x=10, y=20
struct Point p2 = {.y = 20, .x = 10};  // Designated initializers (C99)
```

### Partial Initialization

```c
struct Point p = {10};    // x=10, y=0
```

### Zero-Initialization

```c
struct Point p = {0};     // x=0, y=0
```

### Array of Structures

```c
struct Point points[] = {
    {1, 2},
    {3, 4},
    {5, 6}
};
```

---

## Pointers to Structures

Use the arrow operator (`->`) to access members through a pointer.

```c
struct Point p = {10, 20};
struct Point *ptr = &p;

// Both are equivalent:
(*ptr).x = 30;
ptr->y = 40;
```

**The arrow operator (`->`) is syntactic sugar for `(*ptr).member`.**

---

## Passing Structures to Functions

### Pass by Value

The entire structure is copied. This is safe but inefficient for large structures.

```c
void print_point(struct Point p) {
    printf("(%d, %d)\n", p.x, p.y);
}
```

### Pass by Pointer

Only a pointer is passed. This is efficient and allows modification.

```c
void move_point(struct Point *p, int dx, int dy) {
    p->x += dx;
    p->y += dy;
}
```

### Returning Structures

```c
struct Point add_points(struct Point a, struct Point b) {
    struct Point result;
    result.x = a.x + b.x;
    result.y = a.y + b.y;
    return result;
}
```

---

## Nested Structures

Structures can contain other structures.

```c
struct Rectangle {
    struct Point top_left;
    struct Point bottom_right;
};

struct Rectangle rect = {{0, 0}, {10, 10}};
rect.top_left.x = 5;
```

---

## Self-Referential Structures

A structure can contain a pointer to itself. This is used for linked lists and trees.

```c
struct Node {
    int value;
    struct Node *next;    // Pointer to itself
};
```

---

## Structure Alignment and Padding

Structures may have padding between members to satisfy alignment requirements. This is discussed in detail in [37 — Alignment and Padding](37-alignment.md).

```c
struct Padded {
    char c;    // 1 byte, offset 0
    int i;     // 4 bytes, offset 4 (padding 3 bytes)
};

// sizeof(struct Padded) is 8, not 5
```

---

## Common Pitfalls

### 1. Forgetting the `struct` Keyword

```c
Point p;    // Error if no typedef
```

### 2. Using Dot Instead of Arrow

```c
struct Point *p = &point;
p.x = 10;    // Error: p is a pointer
p->x = 10;   // Correct
```

### 3. Using Arrow Instead of Dot

```c
struct Point p;
p->x = 10;   // Error: p is not a pointer
p.x = 10;    // Correct
```

### 4. Returning a Local Structure

Returning a structure by value is safe (the copy is returned). Returning a pointer to a local structure is dangerous.

```c
struct Point *bad(void) {
    struct Point p = {10, 20};
    return &p;    // Dangerous: p is destroyed
}
```

### 5. Assuming No Padding

```c
sizeof(struct Example) != sum of member sizes
```

### 6. Comparing Structures with `==`

```c
struct Point a = {1, 2};
struct Point b = {1, 2};

if (a == b) {    // Error: cannot compare structures
}
```

**Fix:** Compare members manually or use `memcmp` (but careful with padding).

---

## Complete Example

```c
#include <stdio.h>
#include <string.h>

// Simple structure
typedef struct {
    int x;
    int y;
} Point;

// Nested structure
typedef struct {
    Point top_left;
    Point bottom_right;
} Rectangle;

// Self-referential structure (linked list node)
typedef struct Node {
    int value;
    struct Node *next;
} Node;

// Functions with structures
void print_point(Point p) {
    printf("(%d, %d)\n", p.x, p.y);
}

void move_point(Point *p, int dx, int dy) {
    p->x += dx;
    p->y += dy;
}

Point add_points(Point a, Point b) {
    Point result = {a.x + b.x, a.y + b.y};
    return result;
}

void print_rectangle(Rectangle r) {
    printf("Rect: (%d,%d) to (%d,%d)\n",
           r.top_left.x, r.top_left.y,
           r.bottom_right.x, r.bottom_right.y);
}

// Linked list operations
void print_list(Node *head) {
    Node *current = head;
    while (current) {
        printf("%d -> ", current->value);
        current = current->next;
    }
    printf("NULL\n");
}

void free_list(Node *head) {
    Node *current = head;
    while (current) {
        Node *next = current->next;
        free(current);
        current = next;
    }
}

int main(void) {
    // Basic structure
    Point p1 = {10, 20};
    Point p2 = {30, 40};
    
    printf("p1: ");
    print_point(p1);
    
    // Pointer to structure
    Point *p_ptr = &p1;
    p_ptr->x = 100;
    p_ptr->y = 200;
    printf("p1 after pointer modification: ");
    print_point(p1);
    
    // Moving
    move_point(&p1, 5, 10);
    printf("p1 after move: ");
    print_point(p1);
    
    // Adding
    Point sum = add_points(p1, p2);
    printf("p1 + p2 = ");
    print_point(sum);
    
    // Nested structure
    Rectangle rect = {
        {0, 0},
        {10, 10}
    };
    print_rectangle(rect);
    
    // Array of structures
    Point points[] = {
        {1, 2},
        {3, 4},
        {5, 6}
    };
    printf("points[1] = ");
    print_point(points[1]);
    
    // Linked list
    Node *head = NULL;
    for (int i = 0; i < 5; i++) {
        Node *node = malloc(sizeof(Node));
        if (node) {
            node->value = i * 10;
            node->next = head;
            head = node;
        }
    }
    printf("Linked list: ");
    print_list(head);
    
    // Free list
    free_list(head);
    
    // Size and padding
    printf("\nSize of Point: %zu\n", sizeof(Point));
    printf("Size of Node: %zu\n", sizeof(Node));
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [Project 05 — Dynamic Array Implementation](projects/05-dynamic-array/)
- **Next:** [39 — Structure Padding (`sizeof` lies)](39-struct-padding.md)
- **Pointer to structures:** [40 — Pointers to Structures (`->`)](40-struct-pointers.md)
- **Alignment and padding:** [37 — Alignment and Padding](37-alignment.md)
- **Typedef:** [44 — `typedef`](44-typedef.md)

---

## References

- ISO/IEC 9899:2018 §6.7.2.1 — Structure and union specifiers
- ISO/IEC 9899:2018 §6.7.9 — Initialization
- ISO/IEC 9899:2018 §6.5.2.3 — Structure and union members