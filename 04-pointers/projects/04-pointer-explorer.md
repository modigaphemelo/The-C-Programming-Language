# Part 04 Project: Pointer Explorer

---

## Overview

You have learned how pointers work in C. Now you will build a series of programs that explore and demonstrate pointer concepts. This project will require you to:

- Use pointers in various contexts
- Perform pointer arithmetic
- Work with arrays of pointers
- Use function pointers
- Implement dynamic memory with pointers
- Understand pointer-to-pointer relationships

This project is designed to solidify your understanding of pointers—the most important concept in C.

---

## Project Structure

```
04-pointers/projects/04-pointer-explorer/
├── README.md
├── pointer_basics.c
├── pointer_arithmetic.c
├── pointer_arrays.c
├── function_pointers.c
├── dynamic_2d_array.c
├── linked_list.c
├── Makefile
└── tests/
    └── test_runner.c
```

---

## Task 1: Pointer Basics

**File:** `pointer_basics.c`

### Requirements

Write a program that demonstrates:

1. Declaration of pointers to different types
2. Taking addresses with `&`
3. Dereferencing with `*`
4. Null pointers and their handling
5. Void pointers and casting

### Functions

```c
void demonstrate_basic_pointers(void);
void demonstrate_null_handling(void);
void demonstrate_void_pointers(void);
```

### Expected Output

```
=== Pointer Basics ===
int x = 42
&x = 0x7ffeefbff568
p = &x
*p = 42
*p = 100
x = 100

=== Null Handling ===
p = NULL
p is null, cannot dereference

=== Void Pointers ===
void *vp = &x
*(int *)vp = 100
```

---

## Task 2: Pointer Arithmetic

**File:** `pointer_arithmetic.c`

### Requirements

Write a program that demonstrates:

1. Adding integers to pointers
2. Subtracting integers from pointers
3. Subtracting two pointers
4. Comparing pointers
5. Traversing arrays with pointers

### Functions

```c
void demonstrate_arithmetic(void);
void demonstrate_traversal(void);
void demonstrate_difference(void);
```

### Expected Output

```
=== Pointer Arithmetic ===
arr = [10, 20, 30, 40, 50]
p = arr (points to arr[0])
*p = 10
p++ → *p = 20
p += 2 → *p = 40
q = arr + 4 → *q = 50
q - p = 2 (elements between them)

=== Traversal ===
Traversing with pointer:
10 20 30 40 50

=== Difference ===
Pointer difference: 4
Memory bytes: 16
```

---

## Task 3: Arrays of Pointers

**File:** `pointer_arrays.c`

### Requirements

Write a program that demonstrates:

1. Array of pointers to integers
2. Array of pointers to strings
3. Sorting an array of pointers
4. Searching an array of pointers

### Functions

```c
void demonstrate_int_pointer_array(void);
void demonstrate_string_pointer_array(void);
void sort_strings(char *arr[], int n);
int find_string(char *arr[], int n, const char *target);
```

### Expected Output

```
=== Array of Pointers to Integers ===
Original values: 5, 2, 8, 1, 9
Modified values: 10, 4, 16, 2, 18

=== Array of Pointers to Strings ===
Original: Charlie, Alice, Bob
Sorted: Alice, Bob, Charlie
Found 'Bob' at index 1
'Dave' not found
```

---

## Task 4: Function Pointers

**File:** `function_pointers.c`

### Requirements

Write a program that demonstrates:

1. Pointer to a function
2. Array of function pointers (jump table)
3. Passing function pointers as parameters (callbacks)
4. Using function pointers for operations

### Functions

```c
int add(int a, int b);
int subtract(int a, int b);
int multiply(int a, int b);
int divide(int a, int b);
void apply_operation(int a, int b, int (*op)(int, int));
int get_operation(const char *name);
```

### Expected Output

```
=== Function Pointers ===
add(10, 5) = 15
subtract(10, 5) = 5
multiply(10, 5) = 50
divide(10, 5) = 2

=== Array of Function Pointers ===
add: 15
subtract: 5
multiply: 50
divide: 2

=== Callback ===
apply_operation(6, 3, multiply) = 18

=== Dynamic Dispatch ===
Operation 'add': 15
Operation 'multiply': 50
```

---

## Task 5: Dynamic 2D Array

**File:** `dynamic_2d_array.c`

### Requirements

Write a program that:

1. Allocates a dynamic 2D array using pointer-to-pointer
2. Fills it with values
3. Prints it
4. Frees the memory
5. Resizes the array

### Functions

```c
int **allocate_matrix(int rows, int cols);
void free_matrix(int **matrix, int rows);
void fill_matrix(int **matrix, int rows, int cols);
void print_matrix(int **matrix, int rows, int cols);
int **resize_matrix(int **matrix, int old_rows, int old_cols, 
                    int new_rows, int new_cols);
```

### Expected Output

```
=== Dynamic 2D Array ===
Original 3x4 matrix:
 0  1  2  3
 4  5  6  7
 8  9 10 11

Resized 4x5 matrix:
 0  1  2  3  0
 4  5  6  7  0
 8  9 10 11  0
 0  0  0  0  0
```

---

## Task 6: Linked List

**File:** `linked_list.c`

### Requirements

Write a program that implements a singly linked list using pointers.

### Functions

```c
typedef struct Node {
    int value;
    struct Node *next;
} Node;

Node *create_node(int value);
void insert_front(Node **head, int value);
void insert_back(Node **head, int value);
void insert_at(Node **head, int value, int position);
int remove_front(Node **head);
int remove_back(Node **head);
int remove_at(Node **head, int position);
void print_list(Node *head);
void free_list(Node **head);
int find_value(Node *head, int value);
void reverse_list(Node **head);
```

### Expected Output

```
=== Linked List ===
Insert front: 10, 20, 30
List: 30 -> 20 -> 10 -> NULL

Insert back: 40
List: 30 -> 20 -> 10 -> 40 -> NULL

Insert at position 2: 25
List: 30 -> 20 -> 25 -> 10 -> 40 -> NULL

Remove front: 30
List: 20 -> 25 -> 10 -> 40 -> NULL

Remove back: 40
List: 20 -> 25 -> 10 -> NULL

Find 25: found at position 1
Find 99: not found

Reverse list:
10 -> 25 -> 20 -> NULL
```

---

## Task 7: Test Runner

**File:** `tests/test_runner.c`

### Requirements

Write a test runner that tests all functions:

1. Test pointer basics
2. Test pointer arithmetic
3. Test arrays of pointers
4. Test function pointers
5. Test dynamic 2D arrays
6. Test linked list

### Expected Output

```
=== Running Tests ===
Pointer Basics: PASS
Pointer Arithmetic: PASS
Arrays of Pointers: PASS
Function Pointers: PASS
Dynamic 2D Array: PASS
Linked List: PASS

All tests passed!
```

---

## Task 8: Makefile

```makefile
CC = gcc
CFLAGS = -std=c18 -Wall -Wextra -Werror -I.
TARGETS = pointer_basics pointer_arithmetic pointer_arrays \
          function_pointers dynamic_2d_array linked_list
TEST_TARGET = test_runner

all: $(TARGETS) $(TEST_TARGET)

pointer_basics: pointer_basics.c
	$(CC) $(CFLAGS) -o $@ $^

pointer_arithmetic: pointer_arithmetic.c
	$(CC) $(CFLAGS) -o $@ $^

pointer_arrays: pointer_arrays.c
	$(CC) $(CFLAGS) -o $@ $^

function_pointers: function_pointers.c
	$(CC) $(CFLAGS) -o $@ $^

dynamic_2d_array: dynamic_2d_array.c
	$(CC) $(CFLAGS) -o $@ $^

linked_list: linked_list.c
	$(CC) $(CFLAGS) -o $@ $^

test_runner: tests/test_runner.c pointer_basics.c pointer_arithmetic.c \
             pointer_arrays.c function_pointers.c dynamic_2d_array.c \
             linked_list.c
	$(CC) $(CFLAGS) -o $@ $^

clean:
	rm -f $(TARGETS) $(TEST_TARGET)

test: $(TEST_TARGET)
	./$(TEST_TARGET)

.PHONY: all clean test
```

---

## Checklist

| Task | File | Completed |
|---|---|---|
| Pointer basics | `pointer_basics.c` | [ ] |
| Pointer arithmetic | `pointer_arithmetic.c` | [ ] |
| Arrays of pointers | `pointer_arrays.c` | [ ] |
| Function pointers | `function_pointers.c` | [ ] |
| Dynamic 2D array | `dynamic_2d_array.c` | [ ] |
| Linked list | `linked_list.c` | [ ] |
| Test runner | `tests/test_runner.c` | [ ] |
| Makefile | `Makefile` | [ ] |
| README | `README.md` | [ ] |

---

## Implementation Notes

### Dynamic 2D Array Allocation

```c
int **allocate_matrix(int rows, int cols) {
    int **matrix = malloc(rows * sizeof(int *));
    if (matrix == NULL) {
        return NULL;
    }
    for (int i = 0; i < rows; i++) {
        matrix[i] = malloc(cols * sizeof(int));
        if (matrix[i] == NULL) {
            // Free previously allocated rows
            for (int j = 0; j < i; j++) {
                free(matrix[j]);
            }
            free(matrix);
            return NULL;
        }
    }
    return matrix;
}
```

### Linked List Reversal

```c
void reverse_list(Node **head) {
    Node *prev = NULL;
    Node *current = *head;
    Node *next = NULL;
    
    while (current != NULL) {
        next = current->next;
        current->next = prev;
        prev = current;
        current = next;
    }
    
    *head = prev;
}
```

### Removing an Element

```c
int remove_at(Node **head, int position) {
    if (*head == NULL || position < 0) {
        return -1;
    }
    
    if (position == 0) {
        return remove_front(head);
    }
    
    Node *current = *head;
    for (int i = 0; i < position - 1; i++) {
        if (current->next == NULL) {
            return -1;  // Position out of bounds
        }
        current = current->next;
    }
    
    if (current->next == NULL) {
        return -1;  // Position out of bounds
    }
    
    Node *to_remove = current->next;
    int value = to_remove->value;
    current->next = to_remove->next;
    free(to_remove);
    
    return value;
}
```

---

## What You've Learned

After completing this project, you have demonstrated:

- Pointers to all types (int, char, void, function)
- Pointer arithmetic and array traversal
- Arrays of pointers
- Function pointers and callbacks
- Dynamic 2D arrays with pointer-to-pointer
- Linked lists with pointers
- Memory allocation and freeing
- Pointer safety practices

Pointers are the heart of C. This project has given you the skills to use them effectively and safely.

---

**Next:** [Part 05: Memory Management — Stack vs Heap](/05-memory/notes/31-stack-heap.md)