# Part 02 Project: Function Library

---

## Overview

You have learned how functions work in C. Now you will build a library of utility functions. This project will require you to:

- Write function prototypes in a header file
- Implement functions in a source file
- Use multiple translation units
- Handle function pointers
- Use variadic functions
- Write a test program

This is your first multi-file project. It is also your first step toward building real C libraries.

---

## Project Structure

```
02-functions/projects/02-function-library/
├── README.md
├── math_utils.h
├── math_utils.c
├── string_utils.h
├── string_utils.c
├── file_utils.h
├── file_utils.c
├── utils.h
├── utils.c
├── main.c
└── Makefile
```

---

## Task 1: Math Utilities

**Files:** `math_utils.h`, `math_utils.c`

### Required Functions

| Function | Description |
|---|---|
| `int max(int a, int b)` | Return the larger of two integers |
| `int min(int a, int b)` | Return the smaller of two integers |
| `int abs_int(int x)` | Return the absolute value of an integer |
| `int clamp(int value, int min, int max)` | Clamp value between min and max |
| `int is_even(int x)` | Return 1 if even, 0 if odd |
| `int is_prime(int n)` | Return 1 if prime, 0 if not |
| `int gcd(int a, int b)` | Greatest common divisor (Euclidean algorithm) |
| `int factorial(int n)` | Return n! (recursive) |

### Prototypes

```c
// math_utils.h
#ifndef MATH_UTILS_H
#define MATH_UTILS_H

int max(int a, int b);
int min(int a, int b);
int abs_int(int x);
int clamp(int value, int min, int max);
int is_even(int x);
int is_prime(int n);
int gcd(int a, int b);
int factorial(int n);

#endif
```

### Bonus: Variadic Sum

Add a function that sums a variable number of integers:

```c
int sum(int count, ...);    // Count = number of arguments
// Usage: sum(3, 10, 20, 30) == 60
```

---

## Task 2: String Utilities

**Files:** `string_utils.h`, `string_utils.c`

### Required Functions

| Function | Description |
|---|---|
| `size_t string_length(const char *s)` | Return length of string (no `strlen`) |
| `char *string_copy(char *dest, const char *src)` | Copy string (no `strcpy`) |
| `char *string_concat(char *dest, const char *src)` | Concatenate strings (no `strcat`) |
| `int string_compare(const char *s1, const char *s2)` | Compare strings (no `strcmp`) |
| `char *string_reverse(char *s)` | Reverse a string in-place |
| `int string_is_palindrome(const char *s)` | Check if string is a palindrome |
| `void string_to_upper(char *s)` | Convert to uppercase |
| `void string_to_lower(char *s)` | Convert to lowercase |

### Prototypes

```c
// string_utils.h
#ifndef STRING_UTILS_H
#define STRING_UTILS_H

#include <stddef.h>

size_t string_length(const char *s);
char *string_copy(char *dest, const char *src);
char *string_concat(char *dest, const char *src);
int string_compare(const char *s1, const char *s2);
char *string_reverse(char *s);
int string_is_palindrome(const char *s);
void string_to_upper(char *s);
void string_to_lower(char *s);

#endif
```

**Note:** These functions should not call the standard library string functions. You are writing your own.

---

## Task 3: File Utilities

**Files:** `file_utils.h`, `file_utils.c`

### Required Functions

| Function | Description |
|---|---|
| `int file_exists(const char *path)` | Check if a file exists |
| `size_t file_size(const char *path)` | Return file size in bytes |
| `char *file_read(const char *path)` | Read entire file into a string |
| `int file_write(const char *path, const char *content)` | Write string to file |
| `int file_copy(const char *src, const char *dest)` | Copy a file |
| `int file_count_lines(const char *path)` | Count lines in a file |

### Prototypes

```c
// file_utils.h
#ifndef FILE_UTILS_H
#define FILE_UTILS_H

#include <stddef.h>

int file_exists(const char *path);
size_t file_size(const char *path);
char *file_read(const char *path);
int file_write(const char *path, const char *content);
int file_copy(const char *src, const char *dest);
int file_count_lines(const char *path);

#endif
```

**Note:** Use standard I/O (`fopen`, `fclose`, `fread`, `fwrite`, etc.).

---

## Task 4: Main Program

**File:** `main.c`

### Requirements

Write a test program that demonstrates all functions:

1. Test math utilities with various inputs
2. Test string utilities with various strings
3. Test file utilities with temporary files

**Example Output:**

```
Math Utilities:
max(10, 20) = 20
min(10, 20) = 10
abs_int(-5) = 5
clamp(100, 0, 50) = 50
is_even(4) = 1
is_prime(7) = 1
is_prime(9) = 0
gcd(12, 18) = 6
factorial(5) = 120
sum(5, 1, 2, 3, 4, 5) = 15

String Utilities:
string_length("hello") = 5
string_copy(dest, "hello") = "hello"
string_concat(dest, " world") = "hello world"
string_compare("hello", "hello") = 0
string_reverse("hello") = "olleh"
string_is_palindrome("racecar") = 1
to_upper("hello") = "HELLO"
to_lower("HELLO") = "hello"

File Utilities:
file_exists("test.txt") = 1
file_size("test.txt") = 42
file_read("test.txt") = "Hello, World!"
file_write("test.txt", "New content") = 1
file_copy("test.txt", "copy.txt") = 1
file_count_lines("test.txt") = 1
```

---

## Task 5: Makefile

Write a Makefile that:

- Compiles all `.c` files into object files
- Links them into a single executable
- Supports `make clean`
- Uses `-Wall -Wextra -Werror`

```makefile
CC = gcc
CFLAGS = -std=c18 -Wall -Wextra -Werror -I.
TARGET = program
OBJS = main.o math_utils.o string_utils.o file_utils.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

.PHONY: all clean
```

---

## Checklist

| Task | Files | Completed |
|---|---|---|
| Math utilities | `math_utils.h`, `math_utils.c` | [ ] |
| String utilities | `string_utils.h`, `string_utils.c` | [ ] |
| File utilities | `file_utils.h`, `file_utils.c` | [ ] |
| Main test program | `main.c` | [ ] |
| Makefile | `Makefile` | [ ] |
| README | `README.md` | [ ] |

---

## Checking Your Work

**Compile:**

```bash
make
./program
```

**Check for memory errors:**

```bash
valgrind ./program
```

**Check for warnings:**

```bash
gcc -std=c18 -Wall -Wextra -Werror *.c
```

**Format code:**

```bash
clang-format -i *.c *.h
```

---

## What You've Learned

After completing this project, you have demonstrated:

- Writing and using headers
- Organizing code across multiple files
- Implementing functions with various patterns
- Using recursion (`factorial`, `gcd`)
- Using variadic functions (`sum`)
- Using pointers in string manipulation
- Using function prototypes correctly
- Compiling and linking multiple files
- Writing a Makefile

This is a significant milestone. You have built a library.

---

**Next:** [Part 03: Arrays and Strings](/03-arrays-strings/notes/17-arrays.md)