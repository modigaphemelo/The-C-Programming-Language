# Part 03 Project: String Manipulation Library

---

## Overview

You have learned how arrays and strings work in C. Now you will build a library of string manipulation functions. This project will require you to:

- Work with arrays of characters (strings)
- Use pointer arithmetic
- Implement your own versions of standard string functions
- Handle command-line arguments
- Work with arrays of strings
- Use dynamic memory (optional)

This is your first project focused on text processing. It will deepen your understanding of how strings work in C.

---

## Project Structure

```
03-arrays-strings/projects/03-string-library/
├── README.md
├── str_utils.h
├── str_utils.c
├── main.c
└── Makefile
```

---

## Task 1: String Utilities

**Files:** `str_utils.h`, `str_utils.c`

### Required Functions

| Function | Description |
|---|---|
| `size_t str_len(const char *s)` | Return length of string (no `strlen`) |
| `char *str_copy(char *dest, const char *src)` | Copy string (no `strcpy`) |
| `char *str_cat(char *dest, const char *src)` | Concatenate strings (no `strcat`) |
| `int str_cmp(const char *s1, const char *s2)` | Compare strings (no `strcmp`) |
| `char *str_chr(const char *s, int c)` | Find character (no `strchr`) |
| `char *str_str(const char *haystack, const char *needle)` | Find substring (no `strstr`) |
| `char *str_reverse(char *s)` | Reverse string in-place |
| `int str_is_palindrome(const char *s)` | Check if string is palindrome |
| `void str_to_upper(char *s)` | Convert to uppercase |
| `void str_to_lower(char *s)` | Convert to lowercase |
| `char *str_trim(char *s)` | Remove leading/trailing whitespace |
| `int str_starts_with(const char *s, const char *prefix)` | Check prefix |
| `int str_ends_with(const char *s, const char *suffix)` | Check suffix |

### Prototypes

```c
// str_utils.h
#ifndef STR_UTILS_H
#define STR_UTILS_H

#include <stddef.h>

size_t str_len(const char *s);
char *str_copy(char *dest, const char *src);
char *str_cat(char *dest, const char *src);
int str_cmp(const char *s1, const char *s2);
char *str_chr(const char *s, int c);
char *str_str(const char *haystack, const char *needle);
char *str_reverse(char *s);
int str_is_palindrome(const char *s);
void str_to_upper(char *s);
void str_to_lower(char *s);
char *str_trim(char *s);
int str_starts_with(const char *s, const char *prefix);
int str_ends_with(const char *s, const char *suffix);

#endif
```

---

## Task 2: Main Test Program

**File:** `main.c`

### Requirements

Write a test program that demonstrates all functions with various test cases:

1. Test `str_len` with empty strings and various strings
2. Test `str_copy` and `str_cat` with adequate buffer sizes
3. Test `str_cmp` with equal, less-than, greater-than cases
4. Test `str_chr` with found and not-found cases
5. Test `str_str` with found and not-found cases
6. Test `str_reverse` and `str_is_palindrome`
7. Test `str_to_upper` and `str_to_lower`
8. Test `str_trim` with leading/trailing whitespace
9. Test `str_starts_with` and `str_ends_with`

### Optional: Command-line Interface

Support command-line arguments:

```bash
./program --test string1 string2
./program --upper "hello world"
./program --lower "HELLO WORLD"
./program --reverse "hello"
./program --palindrome "racecar"
./program --compare "hello" "world"
./program --trim "  hello  "
./program --prefix "hello" "he"
./program --suffix "hello" "lo"
```

---

## Task 3: Bonus Functions

### `str_split` — Split String into Tokens

```c
char **str_split(const char *s, const char *delim, int *count);
```

Returns an array of strings, split on delimiters. The caller must free the array.

### `str_join` — Join Strings

```c
char *str_join(char **strings, int count, const char *sep);
```

Joins an array of strings with a separator.

### `str_count` — Count Occurrences

```c
int str_count(const char *s, const char *sub);
```

Counts occurrences of a substring in a string.

### `str_replace` — Replace Substring

```c
char *str_replace(const char *s, const char *old, const char *new);
```

Replaces all occurrences of `old` with `new` in a string. Returns a new string (caller must free).

---

## Task 4: Makefile

```makefile
CC = gcc
CFLAGS = -std=c18 -Wall -Wextra -Werror -I.
TARGET = program
OBJS = main.o str_utils.o

all: $(TARGET)

$(TARGET): $(OBJS)
	$(CC) $(CFLAGS) -o $@ $^

%.o: %.c
	$(CC) $(CFLAGS) -c $< -o $@

clean:
	rm -f $(OBJS) $(TARGET)

test: $(TARGET)
	./$(TARGET) --test "hello" "world"

.PHONY: all clean test
```

---

## Checklist

| Task | File | Completed |
|---|---|---|
| String utilities header | `str_utils.h` | [ ] |
| String utilities implementation | `str_utils.c` | [ ] |
| Main test program | `main.c` | [ ] |
| Makefile | `Makefile` | [ ] |
| README | `README.md` | [ ] |
| Command-line interface (bonus) | `main.c` | [ ] |
| `str_split` (bonus) | `str_utils.c` | [ ] |
| `str_join` (bonus) | `str_utils.c` | [ ] |
| `str_count` (bonus) | `str_utils.c` | [ ] |
| `str_replace` (bonus) | `str_utils.c` | [ ] |

---

## Implementation Notes

### `str_len`

```c
size_t str_len(const char *s) {
    size_t n = 0;
    while (s[n] != '\0') {
        n++;
    }
    return n;
}
```

### `str_copy`

```c
char *str_copy(char *dest, const char *src) {
    char *p = dest;
    while ((*p++ = *src++) != '\0') {
        // Copy
    }
    return dest;
}
```

### `str_cat`

```c
char *str_cat(char *dest, const char *src) {
    char *p = dest + str_len(dest);
    while ((*p++ = *src++) != '\0') {
        // Copy
    }
    return dest;
}
```

### `str_trim`

```c
char *str_trim(char *s) {
    char *end;
    
    // Trim leading space
    while (isspace((unsigned char)*s)) s++;
    
    if (*s == 0) return s;
    
    // Trim trailing space
    end = s + str_len(s) - 1;
    while (end > s && isspace((unsigned char)*end)) end--;
    
    // Write new null terminator
    *(end + 1) = '\0';
    return s;
}
```

### `str_starts_with`

```c
int str_starts_with(const char *s, const char *prefix) {
    while (*prefix) {
        if (*s != *prefix) return 0;
        s++;
        prefix++;
    }
    return 1;
}
```

### `str_split` (Bonus)

```c
char **str_split(const char *s, const char *delim, int *count) {
    char *str = str_copy(malloc(str_len(s) + 1), s);
    // (dynamic allocation required)
    // Implementation using strtok-like approach
    // Return array of strings
}
```

---

## Sample Output

```
Testing str_len:
  str_len("") = 0
  str_len("hello") = 5
  str_len("hello world") = 11

Testing str_copy:
  str_copy(dest, "hello") = "hello"

Testing str_cat:
  str_cat(dest, " world") = "hello world"

Testing str_cmp:
  str_cmp("hello", "hello") = 0
  str_cmp("hello", "world") = -1
  str_cmp("world", "hello") = 1

Testing str_chr:
  str_chr("hello", 'l') = "llo"
  str_chr("hello", 'x') = NULL

Testing str_str:
  str_str("hello world", "world") = "world"
  str_str("hello world", "xyz") = NULL

Testing str_reverse:
  str_reverse("hello") = "olleh"

Testing str_is_palindrome:
  str_is_palindrome("racecar") = 1
  str_is_palindrome("hello") = 0

Testing str_to_upper:
  str_to_upper("hello") = "HELLO"

Testing str_to_lower:
  str_to_lower("HELLO") = "hello"

Testing str_trim:
  str_trim("  hello  ") = "hello"
  str_trim("\t hello \n") = "hello"

Testing str_starts_with:
  str_starts_with("hello", "he") = 1
  str_starts_with("hello", "wo") = 0

Testing str_ends_with:
  str_ends_with("hello", "lo") = 1
  str_ends_with("hello", "he") = 0
```

---

## Checking Your Work

**Compile:**

```bash
make
./program
```

**Check for memory errors:**

```bash
valgrind --leak-check=full ./program
```

**Check with `-Werror`:**

```bash
gcc -std=c18 -Wall -Wextra -Werror -I. -c *.c
```

**Run tests:**

```bash
./program --test "hello" "world"
```

---

## What You've Learned

After completing this project, you have demonstrated:

- Working with character arrays and strings
- Implementing string functions without the standard library
- Using pointer arithmetic
- Handling command-line arguments
- Working with arrays of strings
- Using `const` for read-only strings
- Managing buffer sizes

You have built a complete string manipulation library from scratch. This is a significant milestone.

---

**Next:** [Part 04: Pointers — The Heart of C](/04-pointers/notes/23-pointer-basics.md)