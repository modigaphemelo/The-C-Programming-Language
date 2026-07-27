# 34: Freeing Memory — `free`, Dangling Pointers

---

## Overview

`free` is the function that deallocates memory previously allocated by `malloc`, `calloc`, or `realloc`. It returns the memory to the heap manager for reuse. Failing to free memory causes leaks. Freeing memory incorrectly causes undefined behavior.

**Key characteristics:**

- Takes a pointer returned by `malloc`/`calloc`/`realloc`
- Does not modify the pointer itself (only the memory it points to)
- After `free`, the pointer becomes a dangling pointer
- Must only be called once on each allocation
- Cannot be called on stack variables

---

## The `free` Function

```c
#include <stdlib.h>

void free(void *ptr);
```

**Rules:**

1. `ptr` must have been returned by `malloc`, `calloc`, or `realloc`
2. `ptr` cannot be `NULL` (though `free(NULL)` is safe and does nothing)
3. You cannot free part of an allocation
4. You cannot free the same pointer twice

**Example:**

```c
int *p = malloc(sizeof(int));
if (p) {
    *p = 42;
    free(p);    // Memory is freed
    // p is now a dangling pointer
}
```

---

## Dangling Pointers

A dangling pointer is a pointer that points to memory that has been freed. Using a dangling pointer is undefined behavior.

```c
int *p = malloc(sizeof(int));
*p = 42;
free(p);
*p = 10;    // Undefined behavior (dangling pointer)
```

**Why dangling pointers are dangerous:**

- The memory may be reused for other allocations
- The memory may be returned to the OS
- The program may crash, corrupt data, or behave unpredictably

**Preventing dangling pointers:**

```c
free(p);
p = NULL;    // Now it's a null pointer, not a dangling pointer
```

After setting to `NULL`, any attempt to use `p` will cause a predictable crash (segmentation fault) rather than silent corruption.

---

## Double Free

Freeing the same pointer twice is undefined behavior.

```c
free(p);
free(p);    // Undefined behavior (double free)
```

**Consequences:**

- Corrupted heap metadata
- Program crash
- Security vulnerabilities

**Prevention:**

```c
free(p);
p = NULL;    // Prevents double free
```

---

## Freeing `NULL`

```c
free(NULL);    // Safe: does nothing
```

This is safe and commonly used to simplify cleanup code.

---

## Heap Manager Bookkeeping

When you call `malloc`, the heap manager allocates slightly more memory than requested. It stores metadata (size, flags, etc.) before or after the user memory. When you call `free`, it uses this metadata to return the memory to the free list.

```
+------------------+
| Metadata (size)   |
+------------------+
| Your data         |
+------------------+
```

**Why this matters:**

- If you write past the end of your allocation, you corrupt the metadata
- Corrupted metadata causes `free` to crash or corrupt other allocations
- This is why buffer overflows are dangerous

---

## Common Pitfalls

### 1. Forgetting to Free

```c
void leak(void) {
    int *p = malloc(sizeof(int));
    // Memory is never freed
}
```

### 2. Freeing Stack Memory

```c
int x = 42;
free(&x);    // Error: cannot free stack memory
```

### 3. Freeing a Pointer Not from `malloc`

```c
int arr[10];
free(arr);    // Error: arr is on the stack
```

### 4. Freeing Part of an Allocation

```c
int *p = malloc(10 * sizeof(int));
free(p + 5);    // Error: cannot free part of an allocation
```

### 5. Freeing the Same Pointer Twice

```c
free(p);
free(p);    // Undefined behavior
```

### 6. Using Memory After Free

```c
free(p);
p[0] = 10;    // Undefined behavior
```

### 7. Forgetting to Set to `NULL`

```c
free(p);
if (p != NULL) {    // p is not NULL, but is invalid
    *p = 10;        // Undefined behavior
}
```

### 8. Freeing in the Wrong Order (Circular Dependencies)

```c
struct Node {
    struct Node *next;
};

void free_list(Node *head) {
    Node *current = head;
    while (current) {
        free(current);    // Bad: loses next pointer
        current = current->next;    // current->next is now invalid
    }
}
```

**Correct order:**

```c
void free_list(Node *head) {
    Node *current = head;
    while (current) {
        Node *next = current->next;    // Save before freeing
        free(current);
        current = next;
    }
}
```

---

## Best Practices

### 1. Always Check `malloc` Return Value

```c
int *p = malloc(sizeof(int));
if (p == NULL) {
    // Handle error
}
```

### 2. Set Pointers to `NULL` After Free

```c
free(p);
p = NULL;
```

### 3. Never Free Twice

```c
free(p);
p = NULL;
// Later: free(p);    // Safe: free(NULL) does nothing
```

### 4. Use a Cleanup Pattern

```c
void cleanup(void) {
    free(p);
    free(q);
    free(r);
    p = q = r = NULL;
}
```

### 5. Use Tools to Detect Leaks

- Valgrind (`valgrind --leak-check=full ./program`)
- Address Sanitizer (`-fsanitize=address`)
- Leak sanitizer (`-fsanitize=leak`)

---

## Complete Example

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Structure with dynamically allocated fields
typedef struct {
    char *name;
    int *values;
    int count;
} Person;

void create_person(Person *p, const char *name, int count) {
    p->name = malloc(strlen(name) + 1);
    if (p->name) {
        strcpy(p->name, name);
    }
    
    p->values = malloc(count * sizeof(int));
    if (p->values) {
        for (int i = 0; i < count; i++) {
            p->values[i] = i * 10;
        }
        p->count = count;
    }
}

void free_person(Person *p) {
    free(p->name);     // Free inner allocation
    free(p->values);   // Free inner allocation
    p->name = NULL;
    p->values = NULL;
    p->count = 0;
}

// Linked list with proper free
typedef struct Node {
    int value;
    struct Node *next;
} Node;

Node *create_node(int value) {
    Node *node = malloc(sizeof(Node));
    if (node) {
        node->value = value;
        node->next = NULL;
    }
    return node;
}

void free_list(Node **head) {
    Node *current = *head;
    while (current) {
        Node *next = current->next;    // Save next before freeing
        free(current);
        current = next;
    }
    *head = NULL;
}

int main(void) {
    // Basic free
    int *p = malloc(sizeof(int));
    if (p) {
        *p = 42;
        printf("*p = %d\n", *p);
        free(p);
        p = NULL;    // Prevent dangling
    }
    
    // Freeing NULL (safe)
    free(NULL);
    free(p);    // p is NULL, safe
    
    // Freeing a structure with inner allocations
    Person person;
    create_person(&person, "Alice", 5);
    if (person.name) {
        printf("Name: %s\n", person.name);
        printf("Values: ");
        for (int i = 0; i < person.count; i++) {
            printf("%d ", person.values[i]);
        }
        printf("\n");
    }
    free_person(&person);
    
    // Linked list
    Node *head = NULL;
    for (int i = 0; i < 5; i++) {
        Node *node = create_node(i * 10);
        if (node) {
            node->next = head;
            head = node;
        }
    }
    
    // Print and free
    Node *current = head;
    while (current) {
        printf("%d ", current->value);
        current = current->next;
    }
    printf("\n");
    
    free_list(&head);
    if (head == NULL) {
        printf("List freed successfully\n");
    }
    
    // RAII-like pattern (free on exit)
    int *arr = malloc(10 * sizeof(int));
    if (arr == NULL) {
        return 1;
    }
    // Use arr...
    free(arr);
    arr = NULL;
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [33 — Dynamic Allocation](33-dynamic-allocation.md)
- **Next:** [35 — Memory Leaks](35-memory-leaks.md)
- **Null pointers:** [26 — Null Pointers](26-null.md)
- **Valgrind:** [78 — Valgrind](/12-tools/notes/78-valgrind.md)
- **Address Sanitizer:** [79 — ASan](/12-tools/notes/79-asan.md)

---

## References

- ISO/IEC 9899:2018 §7.22.3.3 — The `free` function
- `man free` — Memory deallocation
- `man valgrind` — Memory debugging tool