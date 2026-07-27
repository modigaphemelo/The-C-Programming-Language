# 41: Nested Structures — Structures Inside Structures

---

## Overview

A nested structure is a structure that contains another structure as a member. This allows you to build complex data types by composing simpler ones. Nested structures are used for:

- Representing hierarchical data
- Creating more complex data types from simpler ones
- Grouping related structures
- Modeling real-world relationships

**Key characteristics:**

- A structure can contain any type, including other structures
- Members are accessed using the dot operator (`.`) or arrow (`->`)
- Nested structures can be initialized using nested braces
- Memory layout follows the same padding rules as any structure

---

## Declaration

### Basic Nested Structure

```c
typedef struct {
    int x;
    int y;
} Point;

typedef struct {
    Point top_left;
    Point bottom_right;
} Rectangle;
```

### Inline Definition

```c
typedef struct {
    struct {
        int x;
        int y;
    } top_left;
    struct {
        int x;
        int y;
    } bottom_right;
} Rectangle;
```

### Multiple Levels

```c
typedef struct {
    int year;
    int month;
    int day;
} Date;

typedef struct {
    char name[50];
    Date birth_date;
    Date hire_date;
} Employee;
```

---

## Accessing Nested Structure Members

Use the dot operator (`.`) to access members, chaining as needed.

```c
Rectangle rect;
rect.top_left.x = 0;
rect.top_left.y = 0;
rect.bottom_right.x = 10;
rect.bottom_right.y = 10;
```

### With Pointers

```c
Rectangle *ptr = &rect;
ptr->top_left.x = 0;
ptr->top_left.y = 0;
ptr->bottom_right.x = 10;
ptr->bottom_right.y = 10;
```

### Chaining Access

```c
// Dot then dot
rect.top_left.x = 5;

// Arrow then dot
ptr->top_left.x = 5;

// Dot then arrow (if top_left were a pointer)
rect.top_left_ptr->x = 5;

// Arrow then arrow (both pointers)
ptr->top_left_ptr->x = 5;
```

---

## Initialization

### Aggregated Initialization

```c
Rectangle rect = {
    {0, 0},    // top_left
    {10, 10}   // bottom_right
};
```

### Designated Initializers (C99)

```c
Rectangle rect = {
    .top_left = {0, 0},
    .bottom_right = {10, 10}
};
```

### Partial Initialization

```c
Rectangle rect = {
    {0, 0}     // bottom_right is zero-initialized
};
```

### Nested Array Initialization

```c
Employee emp = {
    .name = "Alice",
    .birth_date = {1990, 5, 15},
    .hire_date = {2015, 3, 1}
};
```

---

## Passing Nested Structures to Functions

### Pass by Value (Copies Entire Structure)

```c
void print_rect(Rectangle r) {
    printf("Rect: (%d,%d) to (%d,%d)\n",
           r.top_left.x, r.top_left.y,
           r.bottom_right.x, r.bottom_right.y);
}
```

### Pass by Pointer (Efficient, Allows Modification)

```c
void move_rect(Rectangle *r, int dx, int dy) {
    r->top_left.x += dx;
    r->top_left.y += dy;
    r->bottom_right.x += dx;
    r->bottom_right.y += dy;
}
```

### Returning Nested Structures

```c
Rectangle create_rect(int x1, int y1, int x2, int y2) {
    Rectangle r = {
        {x1, y1},
        {x2, y2}
    };
    return r;
}
```

---

## Nested Structures and Pointers

### Structure Containing Pointers to Structures

```c
typedef struct {
    Point *top_left;
    Point *bottom_right;
} Rectangle;

Point tl = {0, 0};
Point br = {10, 10};
Rectangle rect = {&tl, &br};

rect.top_left->x = 5;    // Modifies tl
```

### Structure Containing a Pointer to Itself (Self-Referential)

```c
typedef struct TreeNode {
    int value;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;
```

---

## Arrays of Nested Structures

```c
Rectangle rects[3] = {
    {{0, 0}, {10, 10}},
    {{5, 5}, {15, 15}},
    {{10, 10}, {20, 20}}
};

rects[1].top_left.x = 7;
```

---

## Incomplete Types

Sometimes you need to declare a structure before defining it. This is necessary for mutual references.

```c
// Incomplete declaration (forward declaration)
struct Node;

// Now we can use it
typedef struct {
    int value;
    struct Node *next;    // Points to a Node
} NodeWrapper;

// Full definition
struct Node {
    int value;
    struct Node *next;
};
```

---

## Common Pitfalls

### 1. Forgetting the Nested Structure's Type

```c
Rectangle r;
r.top_left.x = 10;    // OK
r.top_left = {5, 5};  // May not work as expected
```

**Fix:** Use a temporary variable or designated initializers.

### 2. Copying Nested Structures

```c
Rectangle r1 = {{0, 0}, {10, 10}};
Rectangle r2 = r1;    // Copies the entire nested structure (safe)
```

### 3. Pointers to Nested Members

```c
Point *p = &r.top_left;    // OK: p points to top_left
p->x = 20;                 // Modifies r.top_left.x
```

### 4. Returning a Pointer to a Nested Member

```c
Point *get_top_left(Rectangle *r) {
    return &r->top_left;    // Safe: returns pointer to member
}
```

### 5. Nested Structure Size

```c
sizeof(Rectangle) = sizeof(Point) + sizeof(Point) + padding
```

Nested structures follow the same padding rules as any structure.

---

## Complete Example

```c
#include <stdio.h>
#include <string.h>

// Basic structures
typedef struct {
    int x;
    int y;
} Point;

typedef struct {
    int year;
    int month;
    int day;
} Date;

// Nested structures
typedef struct {
    Point top_left;
    Point bottom_right;
} Rectangle;

typedef struct {
    char name[50];
    Date birth_date;
    Date hire_date;
    Point position;
} Employee;

// Function with nested structure
void print_point(const Point *p) {
    printf("(%d, %d)", p->x, p->y);
}

void print_rect(const Rectangle *r) {
    printf("Rect: ");
    print_point(&r->top_left);
    printf(" to ");
    print_point(&r->bottom_right);
    printf("\n");
}

void print_employee(const Employee *e) {
    printf("Name: %s\n", e->name);
    printf("Birth: %d-%02d-%02d\n", e->birth_date.year, e->birth_date.month, e->birth_date.day);
    printf("Hire: %d-%02d-%02d\n", e->hire_date.year, e->hire_date.month, e->hire_date.day);
    printf("Position: ");
    print_point(&e->position);
    printf("\n");
}

void move_rect(Rectangle *r, int dx, int dy) {
    r->top_left.x += dx;
    r->top_left.y += dy;
    r->bottom_right.x += dx;
    r->bottom_right.y += dy;
}

int main(void) {
    // Basic nested struct
    Rectangle r1 = {
        {0, 0},
        {10, 10}
    };
    
    printf("r1: ");
    print_rect(&r1);
    
    move_rect(&r1, 5, 5);
    printf("After move: ");
    print_rect(&r1);
    
    // Using designated initializers
    Rectangle r2 = {
        .top_left = {5, 5},
        .bottom_right = {15, 15}
    };
    printf("\nr2: ");
    print_rect(&r2);
    
    // Employee with nested structures
    Employee emp = {
        .name = "Alice Smith",
        .birth_date = {1990, 5, 15},
        .hire_date = {2015, 3, 1},
        .position = {42, 17}
    };
    
    printf("\nEmployee:\n");
    print_employee(&emp);
    
    // Array of nested structures
    Rectangle rects[3] = {
        {{0, 0}, {10, 10}},
        {{5, 5}, {15, 15}},
        {{10, 10}, {20, 20}}
    };
    
    printf("\nArray of rectangles:\n");
    for (int i = 0; i < 3; i++) {
        printf("rects[%d]: ", i);
        print_rect(&rects[i]);
    }
    
    // Pointers to nested members
    Point *p = &r1.top_left;
    p->x = 100;
    p->y = 100;
    printf("\nr1 after pointer modification: ");
    print_rect(&r1);
    
    // Self-referential struct (tree node)
    typedef struct TreeNode {
        int value;
        struct TreeNode *left;
        struct TreeNode *right;
    } TreeNode;
    
    TreeNode n1 = {10, NULL, NULL};
    TreeNode n2 = {20, NULL, NULL};
    TreeNode root = {15, &n1, &n2};
    
    printf("\nTree root: %d, left: %d, right: %d\n",
           root.value, root.left->value, root.right->value);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [40 — Pointers to Structures (`->`)](40-struct-pointers.md)
- **Next:** [42 — Unions](42-unions.md)
- **Structures:** [38 — Structures (`struct`)](38-structures.md)
- **Typedef:** [44 — `typedef`](44-typedef.md)
- **Self-referential structures:** [40 — Pointers to Structures (`->`)](40-struct-pointers.md)

---

## References

- ISO/IEC 9899:2018 §6.7.2.1 — Structure and union specifiers
- ISO/IEC 9899:2018 §6.7.9 — Initialization
- ISO/IEC 9899:2018 §6.5.2.3 — Structure and union members