# 40: Pointers to Structures — The `->` Operator

---

## Overview

A pointer to a structure is a variable that stores the address of a structure variable. This is the most common way to work with structures in C—passing them to functions, building linked data structures, and avoiding unnecessary copying.

**Key characteristics:**

- Stores the address of a structure
- Access members with the arrow operator (`->`)
- Efficient (only a pointer is copied)
- Allows modification of the original structure
- Essential for linked lists, trees, and graphs

---

## Declaring and Using Pointers to Structures

```c
typedef struct {
    int x;
    int y;
} Point;

Point p = {10, 20};
Point *ptr = &p;    // ptr points to p
```

**Accessing members:**

```c
// Two ways to access members through a pointer:
ptr->x = 30;        // Arrow operator (preferred)
(*ptr).y = 40;      // Dereference + dot (works but verbose)
```

**The `->` operator is syntactic sugar for `(*ptr).member`.** It exists because `(*ptr).member` is cumbersome to type and read.

---

## Why Use Pointers to Structures

### 1. Efficiency

Passing a structure by value copies the entire structure. For large structures, this is expensive.

```c
// Bad: copies the entire struct
void print_point(Point p) {
    printf("(%d, %d)\n", p.x, p.y);
}

// Good: only copies a pointer (4 or 8 bytes)
void print_point(const Point *p) {
    printf("(%d, %d)\n", p->x, p->y);
}
```

### 2. Modifying the Original

Passing by value creates a copy. Modifying the copy does not affect the original.

```c
// Bad: modifies a copy
void move_point(Point p, int dx, int dy) {
    p.x += dx;    // Only modifies local copy
    p.y += dy;
}

// Good: modifies the original
void move_point(Point *p, int dx, int dy) {
    p->x += dx;
    p->y += dy;
}
```

### 3. Building Data Structures

Pointers to structures are required for linked lists, trees, and graphs.

```c
typedef struct Node {
    int value;
    struct Node *next;    // Pointer to the next node
} Node;
```

---

## Passing Structures to Functions

### Pass by Value

```c
void print_point(Point p) {
    printf("(%d, %d)\n", p.x, p.y);
}

Point p = {10, 20};
print_point(p);    // Copies 8 bytes
```

**When to use:**
- Small structures (a few bytes)
- When you don't need to modify the original

### Pass by Pointer

```c
void print_point(const Point *p) {
    printf("(%d, %d)\n", p->x, p->y);
}

Point p = {10, 20};
print_point(&p);    // Copies 4 or 8 bytes
```

**When to use:**
- Large structures
- When you need to modify the original
- When the function should work on the original data

---

## Returning Structures from Functions

### Return by Value (Safe)

```c
Point add_points(Point a, Point b) {
    Point result;
    result.x = a.x + b.x;
    result.y = a.y + b.y;
    return result;    // Returns a copy
}
```

**This is safe** because the result is copied before the function returns.

### Return by Pointer (Dangerous)

```c
Point *get_point(void) {
    Point p = {10, 20};
    return &p;    // Dangerous: p is destroyed!
}
```

**This is dangerous** because `p` is destroyed when the function returns. The pointer points to invalid memory.

### Return a Pointer to Static or Allocated Memory

```c
// Static (safe, but shared)
Point *get_point_static(void) {
    static Point p = {10, 20};
    return &p;    // Safe: p exists for the entire program
}

// Heap (safe, caller must free)
Point *get_point_heap(void) {
    Point *p = malloc(sizeof(Point));
    if (p) {
        p->x = 10;
        p->y = 20;
    }
    return p;    // Safe: caller must free
}
```

---

## Self-Referential Structures

A structure can contain a pointer to itself. This is essential for linked data structures.

```c
typedef struct Node {
    int value;
    struct Node *next;    // Pointer to next node
} Node;
```

**Why `struct Node *next` and not just `Node *next`?**

Because the `typedef` name `Node` is not visible inside the structure definition until the definition is complete. You must use `struct Node` for self-referential pointers.

```c
// This works:
struct Node {
    int value;
    struct Node *next;    // OK
};

// This does NOT work:
typedef struct {
    int value;
    Node *next;           // Error: Node is not defined yet
} Node;
```

---

## Arrays of Pointers to Structures

```c
Point *points[10];    // Array of 10 pointers to Point

for (int i = 0; i < 10; i++) {
    points[i] = malloc(sizeof(Point));
    if (points[i]) {
        points[i]->x = i;
        points[i]->y = i * 2;
    }
}
```

---

## Common Pitfalls

### 1. Using Dot Instead of Arrow

```c
Point *p = &point;
p.x = 10;     // Error: p is a pointer
p->x = 10;    // Correct
```

### 2. Using Arrow Instead of Dot

```c
Point p;
p->x = 10;    // Error: p is not a pointer
p.x = 10;     // Correct
```

### 3. Dereferencing a Null Pointer

```c
Point *p = NULL;
p->x = 10;    // Segmentation fault
```

**Fix:** Always check for NULL.

```c
if (p != NULL) {
    p->x = 10;
}
```

### 4. Returning a Pointer to a Local Structure

```c
Point *get_point(void) {
    Point p = {10, 20};
    return &p;    // Dangerous
}
```

**Fix:** Use `static`, `malloc`, or return by value.

### 5. Forgetting to Allocate Memory

```c
Point *p;         // Uninitialized pointer
p->x = 10;        // Undefined behavior
```

**Fix:** Point to an existing structure or allocate memory.

```c
Point p;
Point *ptr = &p;    // Points to existing structure
ptr->x = 10;        // OK

Point *p2 = malloc(sizeof(Point));    // Allocate memory
if (p2) {
    p2->x = 10;    // OK
    free(p2);
}
```

### 6. Casting Pointer Types

```c
Point p = {10, 20};
int *ptr = (int *)&p;    // Dangerous: type-punning
int x = *ptr;            // Maybe 10, maybe garbage
```

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>

// Point structure
typedef struct {
    int x;
    int y;
} Point;

// Rectangle structure with pointers (demonstrating efficiency)
typedef struct {
    Point *top_left;
    Point *bottom_right;
} Rectangle;

// Node for linked list
typedef struct Node {
    int value;
    struct Node *next;
} Node;

// Function with pointer parameter
void move_point(Point *p, int dx, int dy) {
    p->x += dx;
    p->y += dy;
}

// Function with const pointer (read-only)
void print_point(const Point *p) {
    printf("(%d, %d)\n", p->x, p->y);
}

// Function returning pointer to static (shared)
Point *get_origin_static(void) {
    static Point origin = {0, 0};
    return &origin;
}

// Function returning pointer to heap (caller must free)
Point *create_point(int x, int y) {
    Point *p = malloc(sizeof(Point));
    if (p) {
        p->x = x;
        p->y = y;
    }
    return p;
}

// Self-referential structure functions
Node *create_node(int value) {
    Node *node = malloc(sizeof(Node));
    if (node) {
        node->value = value;
        node->next = NULL;
    }
    return node;
}

void print_list(const Node *head) {
    const Node *current = head;
    while (current) {
        printf("%d -> ", current->value);
        current = current->next;
    }
    printf("NULL\n");
}

void free_list(Node **head) {
    Node *current = *head;
    while (current) {
        Node *next = current->next;
        free(current);
        current = next;
    }
    *head = NULL;
}

int main(void) {
    // Basic pointer to struct
    Point p1 = {10, 20};
    Point *ptr = &p1;
    
    printf("p1: ");
    print_point(&p1);
    
    ptr->x = 30;
    ptr->y = 40;
    printf("After ptr-> modification: ");
    print_point(&p1);
    
    // Function with pointer parameter
    move_point(&p1, 5, 10);
    printf("After move: ");
    print_point(&p1);
    
    // Static pointer
    Point *origin = get_origin_static();
    printf("Origin: ");
    print_point(origin);
    
    // Heap allocation
    Point *p2 = create_point(100, 200);
    if (p2) {
        printf("p2: ");
        print_point(p2);
        free(p2);
    }
    
    // Array of pointers to structs
    Point *points[3];
    for (int i = 0; i < 3; i++) {
        points[i] = malloc(sizeof(Point));
        if (points[i]) {
            points[i]->x = i * 10;
            points[i]->y = i * 20;
        }
    }
    
    printf("\nArray of pointers:\n");
    for (int i = 0; i < 3; i++) {
        printf("points[%d]: ", i);
        print_point(points[i]);
        free(points[i]);
    }
    
    // Linked list
    Node *head = NULL;
    for (int i = 0; i < 5; i++) {
        Node *node = create_node(i * 10);
        if (node) {
            node->next = head;
            head = node;
        }
    }
    
    printf("\nLinked list: ");
    print_list(head);
    free_list(&head);
    printf("After free: ");
    print_list(head);
    
    // Rectangle with pointers
    Point tl = {0, 0};
    Point br = {10, 10};
    Rectangle rect = {&tl, &br};
    printf("\nRectangle: TL ");
    print_point(rect.top_left);
    printf("Rectangle: BR ");
    print_point(rect.bottom_right);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [39 — Structure Padding (`sizeof` lies)](39-struct-padding.md)
- **Next:** [41 — Nested Structures](41-nested-structs.md)
- **Structures:** [38 — Structures (`struct`)](38-structures.md)
- **Dynamic memory:** [33 — Dynamic Allocation](/05-memory/notes/33-dynamic-allocation.md)
- **Linked lists:** [Project 04 — Pointer Explorer](projects/04-pointer-explorer/)
- **Typedef:** [44 — `typedef`](44-typedef.md)

---

## References

- ISO/IEC 9899:2018 §6.5.2.3 — Structure and union members
- ISO/IEC 9899:2018 §6.7.2.1 — Structure and union specifiers
- ISO/IEC 9899:2018 §6.7.9 — Initialization