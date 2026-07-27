# 14: Recursion — Base Cases, Stack Depth

---

## Overview

A recursive function calls itself. This is a powerful technique for problems that can be broken into smaller instances of the same problem. Recursion is not a replacement for iteration—it is an alternative approach, sometimes clearer, sometimes more elegant, sometimes more expensive.

```c
int factorial(int n) {
    if (n <= 1) return 1;              // Base case
    return n * factorial(n - 1);       // Recursive case
}
```

Every recursive function has two essential parts:

| Part | Purpose |
|---|---|
| Base case | Stops the recursion |
| Recursive case | Calls itself with a smaller/simpler input |

Without a base case, the function calls itself forever (until the stack overflows).

---

## How Recursion Works

When a function calls itself, each call creates a new stack frame. The function's local variables and parameters are stored on the stack, and the return address is saved so execution can resume after the call returns.

**Factorial(4) stack trace:**

```
factorial(4)
    n = 4
    calls factorial(3)
        n = 3
        calls factorial(2)
            n = 2
            calls factorial(1)
                n = 1
                returns 1
            returns 2 * 1 = 2
        returns 3 * 2 = 6
    returns 4 * 6 = 24
```

Each call waits for the call below it to return. The stack grows as the recursion goes deeper and shrinks as it unwinds.

---

## Base Cases

The base case is the condition that stops the recursion. It must be reachable.

```c
// Correct: base case n <= 1
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// Incorrect: no base case
int infinite(int n) {
    return infinite(n - 1);    // Infinite recursion
}
```

**Sometimes you need multiple base cases:**

```c
int fibonacci(int n) {
    if (n == 0) return 0;      // Base case 1
    if (n == 1) return 1;      // Base case 2
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

**Sometimes the base case is implicit:**

```c
void print_array(int arr[], int n) {
    if (n == 0) return;        // Base case: empty array
    print_array(arr, n - 1);   // Recursive case
    printf("%d ", arr[n - 1]);
}
```

---

## Recursive vs Iterative

Most recursive functions can be written iteratively (with loops). Which to choose depends on clarity and performance.

| Aspect | Recursion | Iteration |
|---|---|---|
| Clarity (certain problems) | Excellent | Can be complex |
| Stack memory | O(n) per call | O(1) |
| Performance | Slower (function call overhead) | Faster |
| Risk | Stack overflow | No stack overflow |

**When to use recursion:**

- Tree traversal
- Graph traversal (DFS)
- Divide-and-conquer algorithms (merge sort, quicksort)
- Problems with recursive structure (Fibonacci, factorial)
- When the iterative version is significantly more complex

**When to use iteration:**

- Simple counting loops
- When recursion depth is unpredictable
- Performance-critical code
- Embedded systems with limited stack

---

## Tail Recursion

A function is tail-recursive if the recursive call is the last operation before returning. Some compilers can optimize tail recursion into iteration (tail call optimization), eliminating stack growth.

```c
// Not tail-recursive (multiplication happens after the recursive call)
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// Tail-recursive (accumulator carries the result)
int factorial_tail(int n, int acc) {
    if (n <= 1) return acc;
    return factorial_tail(n - 1, n * acc);
}

// Wrapper function
int factorial(int n) {
    return factorial_tail(n, 1);
}
```

**Important:** Tail call optimization is not guaranteed in C. Compilers may or may not perform it. Do not rely on it for deep recursion.

---

## Stack Depth Limitations

Each recursive call consumes stack space. The maximum recursion depth is platform-dependent:

- Linux default: ~8 MB stack
- Windows default: ~1 MB stack
- Embedded systems: much smaller

```c
// This will likely crash with a stack overflow
void deep_recursion(int n) {
    if (n == 0) return;
    deep_recursion(n - 1);
}
```

**Estimate stack usage:** Each frame contains return address, saved registers, local variables, and parameters. A typical frame is 16-64 bytes. On an 8 MB stack, you can have roughly 100,000-500,000 recursive calls.

**Use recursion with bounded depth.** If you are recursing more than a few thousand levels, consider iteration.

---

## Common Recursive Examples

### Factorial

```c
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}
```

### Fibonacci

```c
int fibonacci(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    return fibonacci(n - 1) + fibonacci(n - 2);
}
```

**Note:** This is extremely inefficient (O(2^n) time). The iterative version is much faster.

### String Length

```c
size_t strlen_recursive(const char *s) {
    if (*s == '\0') return 0;
    return 1 + strlen_recursive(s + 1);
}
```

### Print Array in Reverse

```c
void print_reverse(int arr[], int n) {
    if (n == 0) return;
    printf("%d ", arr[n - 1]);
    print_reverse(arr, n - 1);
}
```

### Power Function

```c
double power(double base, int exp) {
    if (exp == 0) return 1.0;
    if (exp < 0) return 1.0 / power(base, -exp);
    return base * power(base, exp - 1);
}
```

---

## Common Pitfalls

### 1. No Base Case

```c
void infinite_recursion(void) {
    infinite_recursion();    // Stack overflow
}
```

### 2. Base Case Not Reachable

```c
int bad_recursion(int n) {
    if (n > 0) return 0;      // This base case doesn't help
    return bad_recursion(n - 1);    // n gets more negative
}
```

### 3. Too Much Work in Each Call

```c
// Quadratic complexity for no reason
void print_pattern(int n) {
    if (n <= 0) return;
    for (int i = 0; i < n; i++) {
        printf("*");
    }
    printf("\n");
    print_pattern(n - 1);
}
```

### 4. Not Trusting the Recursive Call

```c
int sum(int arr[], int n) {
    if (n == 0) return 0;
    return arr[n - 1] + sum(arr, n - 1);    // Trust that sum works
}
```

---

## Complete Example

```c
#include <stdio.h>

// Factorial (recursive)
int factorial(int n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1);
}

// Factorial (tail-recursive)
int factorial_tail(int n, int acc) {
    if (n <= 1) return acc;
    return factorial_tail(n - 1, n * acc);
}

// Fibonacci (recursive)
int fibonacci(int n) {
    if (n == 0) return 0;
    if (n == 1) return 1;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// Print array in reverse (recursive)
void print_reverse(int arr[], int n) {
    if (n == 0) return;
    printf("%d ", arr[n - 1]);
    print_reverse(arr, n - 1);
}

// Tree traversal (binary tree recursion)
typedef struct Node {
    int value;
    struct Node *left;
    struct Node *right;
} Node;

void inorder(Node *root) {
    if (root == NULL) return;
    inorder(root->left);
    printf("%d ", root->value);
    inorder(root->right);
}

int main(void) {
    printf("factorial(5) = %d\n", factorial(5));
    printf("factorial_tail(5) = %d\n", factorial_tail(5, 1));
    printf("fibonacci(10) = %d\n", fibonacci(10));
    
    int arr[] = {1, 2, 3, 4, 5};
    printf("print_reverse: ");
    print_reverse(arr, 5);
    printf("\n");
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [13 — Scope and Lifetime](13-scope.md)
- **Next:** [15 — Inline Functions](15-inline.md)
- **Stack vs heap:** [31 — Stack vs Heap](/05-memory/notes/31-stack-heap.md)
- **Pointers to functions:** [29 — Function Pointers](/04-pointers/notes/29-function-pointers.md)

---

## References

- ISO/IEC 9899:2018 §6.5.2.2 — Function calls
- ISO/IEC 9899:2018 §6.7.6.3 — Function declarators