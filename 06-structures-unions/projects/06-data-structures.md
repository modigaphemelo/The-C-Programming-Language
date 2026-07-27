# Part 06 Project: Data Structure Library

---

## Overview

You have learned how structures, unions, and typedefs work in C. Now you will build a library of common data structures using these concepts. This project will require you to:

- Use structures to organize data
- Use self-referential structures (linked lists, trees)
- Use unions for variant types
- Use typedefs for clean interfaces
- Manage dynamic memory
- Implement common data structures from scratch

This project will demonstrate that you can build real data structures in C.

---

## Project Structure

```
06-structures-unions/projects/06-data-structures/
├── README.md
├── include/
│   ├── list.h
│   ├── stack.h
│   ├── queue.h
│   ├── hashtable.h
│   ├── tree.h
│   └── utils.h
├── src/
│   ├── list.c
│   ├── stack.c
│   ├── queue.c
│   ├── hashtable.c
│   ├── tree.c
│   └── utils.c
├── tests/
│   ├── test_list.c
│   ├── test_stack.c
│   ├── test_queue.c
│   ├── test_hashtable.c
│   ├── test_tree.c
│   └── test_all.c
├── Makefile
└── valgrind.supp (optional)
```

---

## Task 1: Generic Types

**File:** `include/utils.h`, `src/utils.c`

### Required Types

```c
// Generic data type
typedef union {
    int i;
    float f;
    char *s;
    void *ptr;
} Data;

// Data type indicator
typedef enum {
    TYPE_INT,
    TYPE_FLOAT,
    TYPE_STRING,
    TYPE_PTR
} DataType;

// Comparison function type
typedef int (*CompareFn)(const Data *a, const Data *b);

// Print function type
typedef void (*PrintFn)(const Data *d);
```

### Required Functions

```c
Data create_int(int value);
Data create_float(float value);
Data create_string(const char *value);
Data create_ptr(void *value);

void print_int(const Data *d);
void print_float(const Data *d);
void print_string(const Data *d);
void print_ptr(const Data *d);

int compare_int(const Data *a, const Data *b);
int compare_float(const Data *a, const Data *b);
int compare_string(const Data *a, const Data *b);
int compare_ptr(const Data *a, const Data *b);

void free_data(Data *d);
```

---

## Task 2: Singly Linked List

**File:** `include/list.h`, `src/list.c`

### Required Types

```c
typedef struct ListNode {
    Data data;
    struct ListNode *next;
} ListNode;

typedef struct {
    ListNode *head;
    ListNode *tail;
    size_t size;
    CompareFn compare;
    PrintFn print;
} List;
```

### Required Functions

```c
// Creation and destruction
List *list_create(CompareFn compare, PrintFn print);
void list_free(List *list);

// Size and capacity
size_t list_size(const List *list);
int list_empty(const List *list);

// Insertion
int list_push_front(List *list, Data data);
int list_push_back(List *list, Data data);
int list_insert_at(List *list, size_t index, Data data);

// Removal
Data list_pop_front(List *list);
Data list_pop_back(List *list);
Data list_remove_at(List *list, size_t index);

// Access
Data list_get(const List *list, size_t index);
int list_set(List *list, size_t index, Data data);

// Search
int list_find(const List *list, Data data);
int list_contains(const List *list, Data data);

// Iteration
void list_foreach(const List *list, void (*func)(Data *));

// Utility
void list_clear(List *list);
List *list_copy(const List *list);
void list_reverse(List *list);
```

---

## Task 3: Stack

**File:** `include/stack.h`, `src/stack.c`

### Required Types

```c
typedef struct {
    List *list;  // Use the List implementation
} Stack;
```

### Required Functions

```c
Stack *stack_create(PrintFn print);
void stack_free(Stack *s);

int stack_empty(const Stack *s);
size_t stack_size(const Stack *s);

int stack_push(Stack *s, Data data);
Data stack_pop(Stack *s);
Data stack_peek(const Stack *s);

void stack_clear(Stack *s);
void stack_print(const Stack *s);
```

---

## Task 4: Queue

**File:** `include/queue.h`, `src/queue.c`

### Required Types

```c
typedef struct {
    List *list;  // Use the List implementation
} Queue;
```

### Required Functions

```c
Queue *queue_create(PrintFn print);
void queue_free(Queue *q);

int queue_empty(const Queue *q);
size_t queue_size(const Queue *q);

int queue_enqueue(Queue *q, Data data);
Data queue_dequeue(Queue *q);
Data queue_peek(const Queue *q);

void queue_clear(Queue *q);
void queue_print(const Queue *q);
```

---

## Task 5: Hash Table

**File:** `include/hashtable.h`, `src/hashtable.c`

### Required Types

```c
typedef struct HashEntry {
    char *key;
    Data value;
    struct HashEntry *next;
} HashEntry;

typedef struct {
    HashEntry **buckets;
    size_t size;
    size_t capacity;
    CompareFn compare;
    PrintFn print;
} HashTable;
```

### Required Functions

```c
// Creation and destruction
HashTable *hashtable_create(size_t capacity, CompareFn compare, PrintFn print);
void hashtable_free(HashTable *ht);

// Size
size_t hashtable_size(const HashTable *ht);
int hashtable_empty(const HashTable *ht);

// Operations
int hashtable_insert(HashTable *ht, const char *key, Data value);
Data hashtable_get(const HashTable *ht, const char *key);
int hashtable_remove(HashTable *ht, const char *key);
int hashtable_contains(const HashTable *ht, const char *key);

// Utility
void hashtable_clear(HashTable *ht);
void hashtable_print(const HashTable *ht);
```

---

## Task 6: Binary Search Tree

**File:** `include/tree.h`, `src/tree.c`

### Required Types

```c
typedef struct TreeNode {
    Data data;
    struct TreeNode *left;
    struct TreeNode *right;
} TreeNode;

typedef struct {
    TreeNode *root;
    size_t size;
    CompareFn compare;
    PrintFn print;
} Tree;
```

### Required Functions

```c
// Creation and destruction
Tree *tree_create(CompareFn compare, PrintFn print);
void tree_free(Tree *tree);

// Size
size_t tree_size(const Tree *tree);
int tree_empty(const Tree *tree);

// Insertion and removal
int tree_insert(Tree *tree, Data data);
int tree_remove(Tree *tree, Data data);
int tree_contains(const Tree *tree, Data data);

// Traversal
void tree_preorder(const Tree *tree);
void tree_inorder(const Tree *tree);
void tree_postorder(const Tree *tree);

// Utility
int tree_height(const Tree *tree);
int tree_is_balanced(const Tree *tree);
void tree_print(const Tree *tree);
```

---

## Task 7: Test Programs

### Test List

```c
void test_list(void) {
    List *list = list_create(compare_int, print_int);
    
    // Test push
    list_push_back(list, create_int(10));
    list_push_back(list, create_int(20));
    list_push_front(list, create_int(5));
    // List should be: 5, 10, 20
    
    // Test get
    Data d = list_get(list, 1);  // 10
    
    // Test remove
    Data removed = list_remove_at(list, 1);  // 10
    // List should be: 5, 20
    
    // Test find
    int pos = list_find(list, create_int(20));  // 1
    
    // Test iteration
    list_foreach(list, print_int);
    
    list_free(list);
}
```

### Test Stack

```c
void test_stack(void) {
    Stack *s = stack_create(print_int);
    
    stack_push(s, create_int(10));
    stack_push(s, create_int(20));
    stack_push(s, create_int(30));
    
    // Top: 30, 20, 10
    
    Data top = stack_peek(s);  // 30
    Data popped = stack_pop(s); // 30
    
    stack_print(s);  // 20, 10
    
    stack_free(s);
}
```

### Test Queue

```c
void test_queue(void) {
    Queue *q = queue_create(print_int);
    
    queue_enqueue(q, create_int(10));
    queue_enqueue(q, create_int(20));
    queue_enqueue(q, create_int(30));
    
    // Front: 10, 20, 30
    
    Data front = queue_peek(q);  // 10
    Data dequeued = queue_dequeue(q); // 10
    
    queue_print(q);  // 20, 30
    
    queue_free(q);
}
```

### Test Hash Table

```c
void test_hashtable(void) {
    HashTable *ht = hashtable_create(16, compare_int, print_int);
    
    hashtable_insert(ht, "one", create_int(1));
    hashtable_insert(ht, "two", create_int(2));
    hashtable_insert(ht, "three", create_int(3));
    
    Data val = hashtable_get(ht, "two");  // 2
    int exists = hashtable_contains(ht, "two");  // 1
    
    hashtable_remove(ht, "two");
    exists = hashtable_contains(ht, "two");  // 0
    
    hashtable_free(ht);
}
```

### Test Tree

```c
void test_tree(void) {
    Tree *tree = tree_create(compare_int, print_int);
    
    tree_insert(tree, create_int(10));
    tree_insert(tree, create_int(5));
    tree_insert(tree, create_int(15));
    tree_insert(tree, create_int(3));
    tree_insert(tree, create_int(7));
    
    // Tree structure:
    //       10
    //      /  \
    //     5    15
    //    / \
    //   3   7
    
    int contains = tree_contains(tree, create_int(7));  // 1
    int height = tree_height(tree);  // 3
    
    tree_inorder(tree);  // 3, 5, 7, 10, 15
    
    tree_free(tree);
}
```

---

## Task 8: Test All

**File:** `tests/test_all.c`

Run all tests and report results.

```c
int main(void) {
    printf("=== Testing Data Structures ===\n\n");
    
    test_list();
    test_stack();
    test_queue();
    test_hashtable();
    test_tree();
    
    printf("\nAll tests passed!\n");
    return 0;
}
```

---

## Task 9: Makefile

```makefile
CC = gcc
CFLAGS = -std=c18 -Wall -Wextra -Werror -I./include
TARGET = test_all
SRC = src/list.c src/stack.c src/queue.c src/hashtable.c src/tree.c src/utils.c
TEST_SRC = tests/test_list.c tests/test_stack.c tests/test_queue.c \
           tests/test_hashtable.c tests/test_tree.c tests/test_all.c
OBJ = $(SRC:.c=.o) $(TEST_SRC:.c=.o)

all: $(TARGET)

$(TARGET): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJ) $(TARGET)

test: $(TARGET)
	./$(TARGET)

valgrind: $(TARGET)
	valgrind --leak-check=full --show-leak-kinds=all ./$(TARGET)

.PHONY: all clean test valgrind
```

---

## Checklist

| Task | Files | Completed |
|---|---|---|
| Generic types | `include/utils.h`, `src/utils.c` | [ ] |
| Linked list | `include/list.h`, `src/list.c` | [ ] |
| Stack | `include/stack.h`, `src/stack.c` | [ ] |
| Queue | `include/queue.h`, `src/queue.c` | [ ] |
| Hash table | `include/hashtable.h`, `src/hashtable.c` | [ ] |
| Binary search tree | `include/tree.h`, `src/tree.c` | [ ] |
| Test programs | `tests/*.c` | [ ] |
| Makefile | `Makefile` | [ ] |
| README | `README.md` | [ ] |
| Valgrind verification | — | [ ] |

---

## Implementation Notes

### Hash Table Collision Resolution

Use chaining (linked lists in buckets).

```c
int hashtable_insert(HashTable *ht, const char *key, Data value) {
    unsigned int index = hash_function(key) % ht->capacity;
    HashEntry *entry = malloc(sizeof(HashEntry));
    if (!entry) return -1;
    
    entry->key = strdup(key);
    entry->value = value;
    entry->next = ht->buckets[index];
    ht->buckets[index] = entry;
    ht->size++;
    return 0;
}
```

### Hash Function

A simple hash function for strings:

```c
unsigned int hash_function(const char *key) {
    unsigned int hash = 0;
    while (*key) {
        hash = hash * 31 + (*key++);
    }
    return hash;
}
```

### Tree Rebalancing

For the binary search tree, you can skip balancing for this project. Focus on correct insertion, deletion, and traversal.

---

## What You've Learned

After completing this project, you have demonstrated:

- Using structures to organize data
- Using self-referential structures (linked lists, trees)
- Using unions for variant types
- Using typedefs for clean interfaces
- Managing dynamic memory correctly
- Implementing common data structures from scratch
- Using `valgrind` to detect memory leaks
- Writing modular, reusable code

You have built a complete data structure library. This is a significant milestone.

---

**Next:** [Part 07: Input/Output — Standard I/O](/07-io/notes/45-stdio.md)