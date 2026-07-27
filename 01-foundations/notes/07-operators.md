# 07: Operators — Arithmetic, Relational, Logical, Bitwise

---

## Overview

C provides a rich set of operators. Many behave the same way across languages—arithmetic, comparison, assignment. Some behave differently—bitwise, pointer arithmetic, and the surprising aspects of certain relational and logical operators.

Operators are the verbs of the language. They act on data. Understanding how they work is fundamental.

---

## Arithmetic Operators

| Operator | Description | Example |
|---|---|---|
| `+` | Addition | `x + y` |
| `-` | Subtraction | `x - y` |
| `*` | Multiplication | `x * y` |
| `/` | Division | `x / y` |
| `%` | Modulo (remainder) | `x % y` |
| `++` | Increment | `x++` or `++x` |
| `--` | Decrement | `x--` or `--x` |

### Division

Division of integers truncates toward zero (in C99 and later):

```c
int a = 10 / 3;     // 3 (not 3.333)
int b = -10 / 3;    // -3 (truncated toward zero)
```

If either operand is floating-point, the result is floating-point:

```c
float c = 10 / 3.0; // 3.333333
float d = 10.0 / 3; // 3.333333
```

### Modulo

The `%` operator works only on integers. The result has the same sign as the dividend:

```c
int a = 10 % 3;     // 1
int b = -10 % 3;    // -1
int c = 10 % -3;    // 1
```

### Increment and Decrement

`++` and `--` can be used as prefix or postfix:

```c
int x = 5;
int a = ++x;        // x becomes 6, a becomes 6
int b = x++;        // b becomes 6, x becomes 7
```

**Postfix** returns the old value, then increments. **Prefix** increments, then returns the new value.

**Recommendation:** Use `x++` and `--x` sparingly in complex expressions. They are a common source of bugs.

---

## Relational Operators

| Operator | Description | Example |
|---|---|---|
| `==` | Equal to | `x == y` |
| `!=` | Not equal to | `x != y` |
| `<` | Less than | `x < y` |
| `>` | Greater than | `x > y` |
| `<=` | Less than or equal to | `x <= y` |
| `>=` | Greater than or equal to | `x >= y` |

**Caveat:** Floating-point comparisons should use a tolerance, not exact equality:

```c
double a = 0.1 + 0.2;
double b = 0.3;

if (a == b) { }         // Almost certainly false

#include <math.h>
if (fabs(a - b) < 1e-9) { }  // True
```

Relational operators return `1` (true) or `0` (false).

---

## Logical Operators

| Operator | Description | Example |
|---|---|---|
| `&&` | Logical AND | `x && y` |
| `||` | Logical OR | `x || y` |
| `!` | Logical NOT | `!x` |

**Short-circuit evaluation:** `&&` and `||` evaluate left to right and stop as soon as the result is known.

```c
if (x != NULL && x->value == 10) { }  // Safe: x->value is only evaluated if x != NULL
if (flag || function_call()) { }      // function_call() is only called if flag is false
```

This is a common and important feature. Use it.

Logical operators return `1` (true) or `0` (false). Any non-zero value is considered true. `0` is false.

---

## Bitwise Operators

These operate on the individual bits of integers. They are essential for systems programming, flags, and low-level hardware interaction.

| Operator | Description | Example |
|---|---|---|
| `&` | Bitwise AND | `x & y` |
| `|` | Bitwise OR | `x | y` |
| `^` | Bitwise XOR | `x ^ y` |
| `~` | Bitwise NOT (one's complement) | `~x` |
| `<<` | Left shift | `x << n` |
| `>>` | Right shift | `x >> n` |

**Bitwise AND (`&`):**

```c
int x = 0b1010;     // 10
int y = 0b1100;     // 12
int z = x & y;      // 0b1000 (8)
```

**Bitwise OR (`|`):**

```c
int x = 0b1010;     // 10
int y = 0b1100;     // 12
int z = x | y;      // 0b1110 (14)
```

**Bitwise XOR (`^`):**

```c
int x = 0b1010;     // 10
int y = 0b1100;     // 12
int z = x ^ y;      // 0b0110 (6)
```

**Bitwise NOT (`~`):**

```c
int x = 0b1010;     // 10 (binary 1010)
int y = ~x;         // ...11110101 (all bits inverted)
```

**Left shift (`<<`):**

```c
int x = 5;          // 0b0101
int y = x << 1;     // 0b1010 (10)
int z = x << 2;     // 0b10100 (20)
```

**Right shift (`>>`):**

```c
int x = 10;         // 0b1010
int y = x >> 1;     // 0b0101 (5)
int z = x >> 2;     // 0b0010 (2)
```

**Right shift of signed integers:** The behavior depends on the implementation. On most systems, right shift of a signed integer preserves the sign bit (arithmetic shift). Right shift of an unsigned integer is always a logical shift (zeros fill from the left).

**Recommendation:** Use unsigned types for bitwise operations.

---

## Assignment Operators

| Operator | Description | Example |
|---|---|---|
| `=` | Simple assignment | `x = y` |
| `+=` | Add and assign | `x += y` |
| `-=` | Subtract and assign | `x -= y` |
| `*=` | Multiply and assign | `x *= y` |
| `/=` | Divide and assign | `x /= y` |
| `%=` | Modulo and assign | `x %= y` |
| `&=` | Bitwise AND and assign | `x &= y` |
| `|=` | Bitwise OR and assign | `x |= y` |
| `^=` | Bitwise XOR and assign | `x ^= y` |
| `<<=` | Left shift and assign | `x <<= y` |
| `>>=` | Right shift and assign | `x >>= y` |

**Note:** Assignment returns the assigned value, allowing chaining:

```c
int a, b, c;
a = b = c = 5;  // All three are now 5
```

---

## Miscellaneous Operators

### Ternary Conditional Operator (`? :`)

```c
int max = (a > b) ? a : b;    // If a > b, max = a; else max = b
```

This is an expression, not a statement, so it can be used anywhere an expression can be used.

### Comma Operator (`,`)

Evaluates left to right and returns the value of the right operand:

```c
int x = (1, 2, 3);     // x = 3
```

Rarely useful except in `for` loops:

```c
for (int i = 0, j = 10; i < j; i++, j--) {
    // Loop body
}
```

### `sizeof` Operator

Returns the size of a type or expression in bytes:

```c
size_t size = sizeof(int);          // Usually 4
size_t size = sizeof(10);           // Size of int
size_t size = sizeof(array);        // Total bytes of array
```

`sizeof` is evaluated at compile time (except for VLA's). The result is of type `size_t`.

### Address-of Operator (`&`) and Dereference Operator (`*`)

See [23 — Pointer Basics](/04-pointers/notes/23-pointer-basics.md).

---

## Operator Precedence and Associativity

Operators have rules about which operations are performed first. This table shows the important ones:

| Precedence | Operators | Associativity |
|---|---|---|
| Highest | `()`, `[]`, `->`, `.` | Left to right |
| | `++` (post), `--` (post) | Left to right |
| | `+` (unary), `-` (unary), `!`, `~`, `*` (deref), `&` (address), `(type)` | Right to left |
| | `*`, `/`, `%` | Left to right |
| | `+`, `-` | Left to right |
| | `<<`, `>>` | Left to right |
| | `<`, `<=`, `>`, `>=` | Left to right |
| | `==`, `!=` | Left to right |
| | `&` (bitwise) | Left to right |
| | `^` | Left to right |
| | `|` | Left to right |
| | `&&` | Left to right |
| | `||` | Left to right |
| | `?:` | Right to left |
| | `=` (and compound assignments) | Right to left |
| Lowest | `,` | Left to right |

**Use parentheses.** Do not rely on precedence, except for the simplest cases. The compiler will get it right. The next person reading your code may not.

---

## Common Pitfalls

### 1. Using `=` Instead of `==`

```c
int x = 5;
if (x = 10) {    // Assigns 10 to x, then tests if x is non-zero
    // Always true
}
```

Some compilers warn about this. Write `if (10 == x)` to catch errors if you write `10 = x`.

### 2. Bitwise vs Logical Operators

```c
if (flags & 0x04) { }      // Bitwise AND — checks a flag
if (flags && 0x04) { }     // Logical AND — always true if flags and 0x04 are non-zero
```

### 3. Precedence with `&` and `==`

```c
if (x & 1 == 0) { }        // Actually: x & (1 == 0) → x & 0
if ((x & 1) == 0) { }      // Correct
```

### 4. Signed Integer Overflow

```c
int x = INT_MAX;
int y = x + 1;          // Undefined behavior (overflow)
```

Signed integer overflow is undefined. Use unsigned integers if you want wrap-around.

### 5. Floating-Point Equality

```c
if (1.0 / 10.0 * 10.0 == 1.0) { }    // May be false
```

### 6. Dangling `else`

```c
if (x > 0)
    if (y > 0)
        printf("Both positive\n");
else
    printf("x <= 0\n");      // This else attaches to the inner if
```

**Fix:** Use braces.

### 7. Short-Circuit Surprises

```c
int x = 0;
if (x != 0 && 10 / x > 1) { }    // Safe: short-circuit prevents division by zero
if (10 / x > 1 && x != 0) { }    // Unsafe: division by zero may occur
```

---

## Complete Example

```c
#include <stdio.h>
#include <limits.h>
#include <math.h>

int main(void) {
    // Arithmetic
    int a = 10, b = 3;
    printf("10 / 3 = %d\n", a / b);
    printf("10 %% 3 = %d\n", a % b);
    
    // Increment
    int x = 5;
    printf("x = %d\n", x);
    printf("++x = %d\n", ++x);
    printf("x++ = %d\n", x++);
    printf("x = %d\n", x);
    
    // Logical
    int flag = 1;
    if (flag && printf("Flag is set\n")) { }
    
    // Bitwise flags
    unsigned int flags = 0;
    const unsigned int FLAG_A = 0x01;
    const unsigned int FLAG_B = 0x02;
    const unsigned int FLAG_C = 0x04;
    
    flags |= FLAG_A;          // Set flag A
    flags |= FLAG_C;          // Set flag C
    
    if (flags & FLAG_A) {
        printf("Flag A is set\n");
    }
    
    if (flags & FLAG_B) {
        printf("Flag B is set\n");  // Not printed
    }
    
    flags &= ~FLAG_A;         // Clear flag A
    
    // Ternary
    int max = (a > b) ? a : b;
    printf("max = %d\n", max);
    
    // sizeof
    printf("sizeof(int) = %zu\n", sizeof(int));
    printf("sizeof(short) = %zu\n", sizeof(short));
    printf("sizeof(long) = %zu\n", sizeof(long));
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [06 — Constants](06-constants.md)
- **Next:** [08 — Control Flow](08-control-flow.md)
- **Bitwise operators in flags:** Common in embedded and systems programming
- **Precedence:** `man operator` (on some systems)
- **Floating-point comparisons:** [04 — Basic Types](04-basic-types.md)
- **Signed overflow:** [85 — Common Undefined Behavior](/13-ub/notes/85-ub-common.md)

---

## References

- ISO/IEC 9899:2018 §6.5 — Expressions
- ISO/IEC 9899:2018 §6.5.2 — Postfix operators
- ISO/IEC 9899:2018 §6.5.3 — Unary operators
- ISO/IEC 9899:2018 §6.5.5 — Multiplicative operators
- ISO/IEC 9899:2018 §6.5.6 — Additive operators
- ISO/IEC 9899:2018 §6.5.7 — Shift operators
- ISO/IEC 9899:2018 §6.5.8 — Relational operators
- ISO/IEC 9899:2018 §6.5.9 — Equality operators
- ISO/IEC 9899:2018 §6.5.10 — Bitwise AND operator
- ISO/IEC 9899:2018 §6.5.11 — Bitwise XOR operator
- ISO/IEC 9899:2018 §6.5.12 — Bitwise OR operator
- ISO/IEC 9899:2018 §6.5.13 — Logical AND operator
- ISO/IEC 9899:2018 §6.5.14 — Logical OR operator
- ISO/IEC 9899:2018 §6.5.15 — Conditional operator
- ISO/IEC 9899:2018 §6.5.16 — Assignment operators
- ISO/IEC 9899:2018 §6.5.17 — Comma operator