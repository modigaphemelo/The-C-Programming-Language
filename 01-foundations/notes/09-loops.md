# 09: Loops — `for`, `while`, `do-while`

---

## Overview

Loops repeat a block of code while a condition holds. C provides three loop constructs:

| Loop | When to Use |
|---|---|
| `while` | When the number of iterations is unknown, depends on a condition |
| `do-while` | When the body must execute at least once |
| `for` | When the number of iterations is known (counting) or involves initialization and update |

Each loop has its place. Choosing the right one makes code clearer and less error-prone.

---

## The `while` Loop

The simplest loop. The condition is tested before each iteration. If false initially, the body never executes.

```c
while (condition) {
    // Body
}
```

**Example:**

```c
int i = 0;
while (i < 10) {
    printf("%d\n", i);
    i++;
}
```

**Use `while` when:**

- The number of iterations is not known in advance
- You are waiting for a condition to become true
- Reading input until EOF
- Traversing a linked list

---

## The `do-while` Loop

The condition is tested after the body executes. The body always runs at least once.

```c
do {
    // Body
} while (condition);
```

**Example:**

```c
int i = 0;
do {
    printf("%d\n", i);
    i++;
} while (i < 10);
```

**Use `do-while` when:**

- The body must execute at least once
- You want to avoid duplicating code that checks a condition
- Validating user input

**Example: Input validation:**

```c
int value;
do {
    printf("Enter a positive number: ");
    scanf("%d", &value);
} while (value <= 0);
```

---

## The `for` Loop

The `for` loop bundles initialization, condition, and update into a single line. This makes it the clearest choice for counting loops.

```c
for (initialization; condition; update) {
    // Body
}
```

**Execution order:**

1. Initialization (executed once)
2. Test condition
3. If true, execute body
4. Execute update
5. Go to step 2

**Example:**

```c
for (int i = 0; i < 10; i++) {
    printf("%d\n", i);
}
```

**Omitting parts:**

```c
// Infinite loop
for (;;) {
    // Body
}

// Initialization outside
int i = 0;
for (; i < 10; i++) {
    printf("%d\n", i);
}

// No body (delay loop)
for (volatile int i = 0; i < 1000000; i++) {
    // Wait
}
```

**Multiple variables:**

```c
for (int i = 0, j = 10; i < j; i++, j--) {
    printf("i = %d, j = %d\n", i, j);
}
```

**Use `for` when:**

- Counting loops (known number of iterations)
- Iterating over arrays
- Any loop with a clear initialization, condition, and update

---

## Break and Continue

### `break`

Exits the loop immediately. Control jumps to the first statement after the loop.

```c
for (int i = 0; i < 10; i++) {
    if (i == 5) {
        break;          // Exit loop when i == 5
    }
    printf("%d\n", i);  // Prints 0-4
}
```

### `continue`

Skips the rest of the current iteration and jumps to the next.

```c
for (int i = 0; i < 10; i++) {
    if (i % 2 == 0) {
        continue;       // Skip even numbers
    }
    printf("%d\n", i);  // Prints odd numbers only
}
```

**Important:** In a `while` or `do-while` loop, `continue` jumps to the condition check. In a `for` loop, it jumps to the update.

---

## Infinite Loops

```c
while (1) {
    // Body
}

for (;;) {
    // Body
}
```

Infinite loops are useful for:

- Server main loops
- Event loops
- Menu systems
- Hardware polling

Always ensure there is a way to exit (usually `break` or `return`).

---

## Nested Loops

Loops can be nested inside other loops:

```c
for (int i = 0; i < 3; i++) {
    for (int j = 0; j < 3; j++) {
        printf("(%d, %d)\n", i, j);
    }
}
```

**Labeled break and continue:**

C does not have labeled breaks (like Java). To break out of nested loops, use a flag or `goto`:

```c
int done = 0;
for (int i = 0; i < 10 && !done; i++) {
    for (int j = 0; j < 10; j++) {
        if (condition) {
            done = 1;
            break;
        }
    }
}
```

```c
// Or use goto (acceptable for breaking nested loops)
for (int i = 0; i < 10; i++) {
    for (int j = 0; j < 10; j++) {
        if (condition) {
            goto done;
        }
    }
}
done:
    // Continue here
```

---

## Common Pitfalls

### 1. Off-by-One Errors

```c
int arr[10];
for (int i = 0; i <= 10; i++) {    // Off-by-one: i goes to 10, out of bounds
    arr[i] = 0;
}
```

**Fix:** Use `<` not `<=` for zero-based arrays.

### 2. Infinite Loops

```c
int i = 0;
while (i < 10) {
    printf("%d\n", i);
    // Missing i++
}
```

### 3. Semicolon After `while` or `for`

```c
while (i < 10); {   // Empty loop body, then a block
    // This block executes once, after the loop finishes
}
```

```c
for (int i = 0; i < 10; i++); {   // Same issue
    // This block executes once, after the loop finishes
}
```

### 4. Modifying the Loop Variable

```c
for (int i = 0; i < 10; i++) {
    if (condition) {
        i = 0;      // Resets the loop, may cause infinite loop
    }
}
```

### 5. Float Loop Conditions

```c
for (float f = 0.0f; f < 1.0f; f += 0.1f) {
    // May run more or fewer times than expected due to floating-point precision
}
```

**Use integers for loop control.**

---

## Loop Selection Summary

| Situation | Recommended Loop |
|---|---|
| Counting known iterations | `for` |
| Reading until EOF | `while` |
| Menu or input validation | `do-while` |
| Traversing a list | `while` or `for` |
| Waiting for a condition | `while` |
| Hardware polling | `while (1)` with `break` |

---

## Complete Example

```c
#include <stdio.h>

int main(void) {
    // while loop
    printf("while loop:\n");
    int i = 0;
    while (i < 5) {
        printf("%d ", i);
        i++;
    }
    printf("\n");
    
    // do-while loop
    printf("do-while loop:\n");
    int j = 0;
    do {
        printf("%d ", j);
        j++;
    } while (j < 5);
    printf("\n");
    
    // for loop
    printf("for loop:\n");
    for (int k = 0; k < 5; k++) {
        printf("%d ", k);
    }
    printf("\n");
    
    // Nested loops (2D array)
    int matrix[3][3] = {
        {1, 2, 3},
        {4, 5, 6},
        {7, 8, 9}
    };
    
    printf("Matrix:\n");
    for (int row = 0; row < 3; row++) {
        for (int col = 0; col < 3; col++) {
            printf("%d ", matrix[row][col]);
        }
        printf("\n");
    }
    
    // break and continue
    printf("break and continue:\n");
    for (int n = 0; n < 10; n++) {
        if (n == 3) {
            continue;   // Skip 3
        }
        if (n == 7) {
            break;      // Exit at 7
        }
        printf("%d ", n);
    }
    printf("\n");
    
    return 0;
}
```

---

## Project: Hello World and Variations

Now is the time to write your first project. In [01-foundations/projects/01-hello-world/](01-foundations/projects/01-hello-world/), create:

1. `hello.c` — Basic Hello World
2. `hello_args.c` — Hello World with command-line arguments
3. `hello_loop.c` — Hello World printed 10 times using each loop type
4. `hello_input.c` — Hello World that asks for your name

---

## Cross-References

- **Previous:** [08 — Control Flow](08-control-flow.md)
- **Next:** [10 — Function Syntax](10-function-syntax.md)
- **Arrays:** See [17 — Arrays](03-arrays-strings/notes/17-arrays.md)
- **Input validation:** See [47 — scanf](07-io/notes/47-scanf.md)
- **Goto:** See [77 — GDB](12-tools/notes/77-gdb.md)

---

## References

- ISO/IEC 9899:2018 §6.8.5 — Iteration statements
- ISO/IEC 9899:2018 §6.8.5.1 — The `while` statement
- ISO/IEC 9899:2018 §6.8.5.2 — The `do` statement
- ISO/IEC 9899:2018 §6.8.5.3 — The `for` statement