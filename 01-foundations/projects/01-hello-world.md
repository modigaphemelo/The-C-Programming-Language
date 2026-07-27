# Part 01 Project: Hello World and Variations

---

## Overview

This project is your first milestone. It exists to demonstrate that you can:

1. Write a C program from scratch
2. Compile and run it
3. Understand the structure of `main()`
4. Handle command-line arguments
5. Use control flow and loops
6. Write and call functions
7. Handle basic input

**There is no single "correct" solution.** Write code that works, and write it cleanly.

---

## Project Structure

```
01-foundations/projects/01-hello-world/
├── README.md
├── hello.c
├── hello_args.c
├── hello_loop.c
├── hello_input.c
└── Makefile (optional)
```

---

## Task 1: Basic Hello World

**File:** `hello.c`

**Requirements:**

- Print `"Hello, World!"` to stdout
- Return `0` (success)

**Expected Output:**

```
Hello, World!
```

**Compile:**

```bash
gcc -std=c18 -Wall -Wextra -Werror hello.c -o hello
./hello
```

---

## Task 2: Hello World with Command-Line Arguments

**File:** `hello_args.c`

**Requirements:**

- If no arguments are provided, print `"Hello, World!"`
- If arguments are provided, print `"Hello, [argument]!"` for each argument

**Expected Output:**

```bash
./hello_args
Hello, World!

./hello_args Alice Bob Charlie
Hello, Alice!
Hello, Bob!
Hello, Charlie!
```

**Concepts tested:**

- `main(int argc, char *argv[])`
- Looping over `argv`
- Conditional logic

---

## Task 3: Hello World with Loops

**File:** `hello_loop.c`

**Requirements:**

- Print `"Hello, World!"` 10 times using:
  - A `while` loop
  - A `do-while` loop
  - A `for` loop
- Each section should be clearly labeled

**Expected Output:**

```
while loop:
1: Hello, World!
2: Hello, World!
...
10: Hello, World!

do-while loop:
1: Hello, World!
2: Hello, World!
...
10: Hello, World!

for loop:
1: Hello, World!
2: Hello, World!
...
10: Hello, World!
```

**Concepts tested:**

- All three loop types
- Output formatting with line numbers

---

## Task 4: Hello World with Input

**File:** `hello_input.c`

**Requirements:**

- Prompt the user for their name
- Read the name from stdin
- Print `"Hello, [name]!"`
- If the user enters an empty line, print `"Hello, World!"`

**Expected Output:**

```
Enter your name: Alice
Hello, Alice!

Enter your name:
Hello, World!
```

**Concepts tested:**

- `fgets()` for safe input
- Handling newlines and empty input
- Basic string operations

---

## Bonus (Optional)

### Bonus 1: Hello World with a Function

Extract the greeting logic into a function:

```c
void greet(const char *name) {
    printf("Hello, %s!\n", name);
}
```

### Bonus 2: Hello World with an Enum

Define an enum for the greeting type (formal, casual, etc.) and print accordingly:

```c
enum GreetingStyle {
    GREETING_FORMAL,
    GREETING_CASUAL,
    GREETING_SHORT
};
```

### Bonus 3: Hello World with a Makefile

Write a simple Makefile that compiles all four programs:

```makefile
CC = gcc
CFLAGS = -std=c18 -Wall -Wextra -Werror

all: hello hello_args hello_loop hello_input

hello: hello.c
	$(CC) $(CFLAGS) -o $@ $^

hello_args: hello_args.c
	$(CC) $(CFLAGS) -o $@ $^

hello_loop: hello_loop.c
	$(CC) $(CFLAGS) -o $@ $^

hello_input: hello_input.c
	$(CC) $(CFLAGS) -o $@ $^

clean:
	rm -f hello hello_args hello_loop hello_input

.PHONY: all clean
```

---

## Checklist

| Task | File | Completed |
|---|---|---|
| Basic Hello World | `hello.c` | [ ] |
| Command-line arguments | `hello_args.c` | [ ] |
| Loops | `hello_loop.c` | [ ] |
| Input | `hello_input.c` | [ ] |
| Function extraction (bonus) | — | [ ] |
| Enum (bonus) | — | [ ] |
| Makefile (bonus) | `Makefile` | [ ] |

---

## Checking Your Work

**Compile with warnings enabled:**

```bash
gcc -std=c18 -Wall -Wextra -Werror hello.c -o hello
```

**Check for memory errors:** (after Part 05)

```bash
valgrind ./hello
```

**Format code before committing:** (after Part 12)

```bash
clang-format -i hello.c
```

---

## What You've Learned

After completing this project, you have demonstrated:

- Program structure (`main()`, headers, return codes)
- Compilation (the four stages, even if you didn't think about them)
- Variables and types (`int`, `char *`, strings)
- Constants (`"Hello, World!"` is a string literal)
- Operators (assignment, arithmetic, comparison)
- Control flow (`if`, `else`, `switch` not used here, but available)
- Loops (`while`, `do-while`, `for`)
- Functions (`main()` is a function, and you wrote one in the bonus)
- Input/output (`printf`, `fgets`)
- Command-line arguments (`argc`, `argv`)

That is everything from Part 01. You have used every concept.

---

**Next:** [Part 02: Functions — The Building Blocks](/02-functions/notes/10-function-syntax.md)