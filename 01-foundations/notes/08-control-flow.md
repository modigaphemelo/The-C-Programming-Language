# 08: Control Flow — `if`, `else`, `switch`

---

## Overview

Control flow statements determine the order in which code executes. Without them, programs run in a straight line from top to bottom. With them, programs can make decisions, repeat operations, and branch conditionally.

C provides two primary control flow constructs for decision-making: `if` and `switch`.

---

## The `if` Statement

The simplest conditional. If the expression evaluates to true (non-zero), the block executes.

```c
if (condition) {
    // Executes if condition is true
}
```

**Example:**

```c
int x = 10;
if (x > 0) {
    printf("x is positive\n");
}
```

### `if-else`

```c
if (condition) {
    // Executes if condition is true
} else {
    // Executes if condition is false
}
```

**Example:**

```c
int x = -5;
if (x > 0) {
    printf("x is positive\n");
} else {
    printf("x is not positive\n");
}
```

### `if-else if-else`

```c
if (condition1) {
    // First condition
} else if (condition2) {
    // Second condition
} else if (condition3) {
    // Third condition
} else {
    // Default
}
```

**Example:**

```c
int score = 85;

if (score >= 90) {
    printf("A\n");
} else if (score >= 80) {
    printf("B\n");
} else if (score >= 70) {
    printf("C\n");
} else if (score >= 60) {
    printf("D\n");
} else {
    printf("F\n");
}
```

---

## The `switch` Statement

`switch` is used when a variable is compared to multiple constant values. It is often clearer than a long chain of `if-else if` statements.

```c
switch (expression) {
    case constant1:
        // Code
        break;
    case constant2:
        // Code
        break;
    default:
        // Code
        break;
}
```

**Key points:**

- The expression must be an integer type (`int`, `char`, `enum`, etc.)
- Each `case` must be a constant integer expression
- `break` exits the switch; without it, execution "falls through" to the next case
- `default` is optional but recommended

**Example:**

```c
int day = 3;

switch (day) {
    case 1:
        printf("Monday\n");
        break;
    case 2:
        printf("Tuesday\n");
        break;
    case 3:
        printf("Wednesday\n");
        break;
    case 4:
        printf("Thursday\n");
        break;
    case 5:
        printf("Friday\n");
        break;
    case 6:
        printf("Saturday\n");
        break;
    case 7:
        printf("Sunday\n");
        break;
    default:
        printf("Invalid day\n");
        break;
}
```

### Fall-Through

Without `break`, execution continues into the next case:

```c
switch (grade) {
    case 'A':
    case 'B':
    case 'C':
        printf("Pass\n");
        break;
    case 'D':
    case 'F':
        printf("Fail\n");
        break;
    default:
        printf("Invalid grade\n");
        break;
}
```

Fall-through is sometimes useful, but it is a common source of bugs. Use it intentionally and comment it clearly.

---

## Conditional Operator (`?:`)

The ternary operator is a compact form of `if-else` for expressions:

```c
result = (condition) ? value_if_true : value_if_false;
```

**Example:**

```c
int max = (a > b) ? a : b;
```

This is not a control flow statement—it is an expression. Use it for simple conditions where readability improves. Avoid nesting ternaries.

---

## Scoping in Control Flow

Variables declared inside a block are scoped to that block:

```c
if (x > 0) {
    int y = x * 2;      // y exists only inside this block
}
// y is not accessible here
```

**C99** allows declarations inside `for` and `if` conditions:

```c
if (int x = get_value(); x > 0) {
    // x is scoped to the if and else blocks
} else {
    // x is still visible here
}
// x is not visible here
```

---

## Common Pitfalls

### 1. Dangling Else

```c
if (x > 0)
    if (y > 0)
        printf("Both positive\n");
else
    printf("x <= 0\n");      // This else attaches to the inner if
```

**Fix:** Use braces.

```c
if (x > 0) {
    if (y > 0) {
        printf("Both positive\n");
    }
} else {
    printf("x <= 0\n");
}
```

### 2. Using `=` Instead of `==`

```c
if (x = 10) {    // Assigns 10 to x, always true
    // Always executes
}
```

**Fix:** Write constants on the left:

```c
if (10 == x) {   // Compiler error if you write "10 = x"
}
```

### 3. Missing `break` in `switch`

```c
switch (x) {
    case 1:
        printf("One\n");    // Falls through to case 2
    case 2:
        printf("Two\n");
        break;
}
```

### 4. Non-Integer `switch` Expression

```c
switch (x) {    // x must be an integer type
    case 1: ...
}
```

Floating-point types are not allowed. Strings are not allowed.

### 5. Duplicate Case Labels

```c
switch (x) {
    case 1:
        break;
    case 1:      // Error: duplicate case value
        break;
}
```

---

## Style Recommendations

1. **Always use braces** for `if`, `else`, `while`, `for`—even for single statements. It prevents errors and makes maintenance easier.

2. **Always include a `default` case in `switch`**—even if empty. It signals that you considered the default case.

3. **Comment intentional fall-through** in `switch`:

```c
switch (x) {
    case 1:
        // fall through
    case 2:
        // ...
        break;
}
```

4. **Keep conditions simple.** If a condition is complex, assign it to a variable:

```c
int is_valid = (x > 0 && y > 0 && z > 0);
if (is_valid) {
    // ...
}
```

5. **Use `switch` when comparing a single variable to multiple constant values.** Use `if` for everything else.

---

## Complete Example

```c
#include <stdio.h>

int main(void) {
    int value = 5;
    char grade = 'B';
    
    // Simple if
    if (value > 0) {
        printf("value is positive\n");
    }
    
    // if-else
    if (value % 2 == 0) {
        printf("value is even\n");
    } else {
        printf("value is odd\n");
    }
    
    // if-else if-else
    if (value > 10) {
        printf("value > 10\n");
    } else if (value > 5) {
        printf("value > 5\n");
    } else {
        printf("value <= 5\n");
    }
    
    // Ternary
    const char *msg = (value > 0) ? "positive" : "non-positive";
    printf("value is %s\n", msg);
    
    // Switch with fall-through
    switch (grade) {
        case 'A':
        case 'B':
        case 'C':
            printf("Pass\n");
            break;
        case 'D':
        case 'F':
            printf("Fail\n");
            break;
        default:
            printf("Invalid grade\n");
            break;
    }
    
    // C99: declaration in if
    if (int x = 42; x > 0) {
        printf("x = %d\n", x);
    }
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [07 — Operators](07-operators.md)
- **Next:** [09 — Loops](09-loops.md)
- **Truthiness:** In C, any non-zero value is true; zero is false
- **`switch` with enums:** Common pattern for state machines
- **Scoping rules:** See [05 — Variables](05-variables.md)

---

## References

- ISO/IEC 9899:2018 §6.8.4 — Selection statements
- ISO/IEC 9899:2018 §6.8.4.1 — The `if` statement
- ISO/IEC 9899:2018 §6.8.4.2 — The `switch` statement