# 20: String Functions — `<string.h>` Library

---

## Overview

The C standard library provides a comprehensive set of string manipulation functions in `<string.h>`. These functions operate on null-terminated strings. They are efficient, well-tested, and widely used.

**Key categories:**

- Length and comparison
- Copying and concatenation
- Searching and tokenization
- Character classification and conversion (in `<ctype.h>`)

---

## Length and Comparison

### `strlen` — String Length

```c
#include <string.h>

size_t strlen(const char *s);
```

Returns the number of characters before the null terminator. The null terminator is not counted.

```c
char str[] = "Hello";
size_t len = strlen(str);    // 5
```

**Time complexity:** O(n) — traverses the entire string.

### `strcmp` — String Compare

```c
int strcmp(const char *s1, const char *s2);
```

Returns 0 if the strings are equal, a negative value if s1 < s2, and a positive value if s1 > s2 (lexicographically, based on ASCII values).

```c
strcmp("Hello", "Hello") == 0
strcmp("Hello", "World") < 0
strcmp("Hello", "Apple") > 0
```

### `strncmp` — String Compare (with length limit)

```c
int strncmp(const char *s1, const char *s2, size_t n);
```

Compares up to `n` characters. Useful for prefix checks:

```c
strncmp("Hello", "Help", 3) == 0   // "Hel" == "Hel"
strncmp("Hello", "World", 3) != 0  // "Hel" != "Wor"
```

---

## Copying and Concatenation

### `strcpy` — Copy String

```c
char *strcpy(char *dest, const char *src);
```

Copies the entire string from `src` to `dest`, including the null terminator.

```c
char dest[20];
strcpy(dest, "Hello");
```

**Warning:** No bounds checking. If `dest` is too small, buffer overflow occurs.

### `strncpy` — Copy String (with length limit)

```c
char *strncpy(char *dest, const char *src, size_t n);
```

Copies up to `n` characters from `src` to `dest`.

```c
char dest[10];
strncpy(dest, "Hello World", 9);
dest[9] = '\0';    // MUST ensure null termination
```

**Important:** `strncpy` does NOT guarantee null termination. If `src` has `n` or more characters, the result is not null-terminated.

### `strcat` — Concatenate Strings

```c
char *strcat(char *dest, const char *src);
```

Appends `src` to the end of `dest`.

```c
char dest[20] = "Hello";
strcat(dest, " World");    // dest = "Hello World"
```

**Warning:** No bounds checking. If `dest` is too small, buffer overflow occurs.

### `strncat` — Concatenate (with length limit)

```c
char *strncat(char *dest, const char *src, size_t n);
```

Appends up to `n` characters from `src` to `dest`. Always null-terminates the result.

```c
char dest[20] = "Hello";
strncat(dest, " World!", 6);    // dest = "Hello World"
```

**Use `strncat` to prevent buffer overflow.**

---

## Searching

### `strchr` — Find Character in String

```c
char *strchr(const char *s, int c);
```

Returns a pointer to the first occurrence of `c` in `s`, or `NULL` if not found.

```c
char *pos = strchr("Hello", 'l');    // Points to the first 'l'
if (pos) {
    printf("Found at: %zu\n", pos - "Hello");    // 2
}
```

### `strrchr` — Find Character from End

```c
char *strrchr(const char *s, int c);
```

Returns a pointer to the last occurrence of `c` in `s`, or `NULL` if not found.

```c
char *pos = strrchr("Hello", 'l');    // Points to the last 'l'
if (pos) {
    printf("Found at: %zu\n", pos - "Hello");    // 3
}
```

### `strstr` — Find Substring

```c
char *strstr(const char *haystack, const char *needle);
```

Returns a pointer to the first occurrence of `needle` in `haystack`, or `NULL` if not found.

```c
char *pos = strstr("Hello World", "World");
if (pos) {
    printf("Found at: %zu\n", pos - "Hello World");    // 6
}
```

### `strpbrk` — Find Any Character from Set

```c
char *strpbrk(const char *s, const char *accept);
```

Returns a pointer to the first character in `s` that matches any character in `accept`.

```c
char *pos = strpbrk("Hello", "aeiou");    // Points to 'e'
```

### `strspn` — Span of Characters in Set

```c
size_t strspn(const char *s, const char *accept);
```

Returns the length of the initial segment of `s` consisting only of characters from `accept`.

```c
size_t len = strspn("Hello123", "Hello");    // 5
```

### `strcspn` — Span of Characters Not in Set

```c
size_t strcspn(const char *s, const char *reject);
```

Returns the length of the initial segment of `s` consisting of characters not in `reject`.

```c
size_t len = strcspn("Hello123", "0123456789");    // 5
```

---

## Tokenization

### `strtok` — Tokenize String

```c
char *strtok(char *str, const char *delim);
```

Splits a string into tokens based on delimiters.

```c
char str[] = "Hello,World,Test";
char *token = strtok(str, ",");
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, ",");
}
```

**Important:** Modifies the input string (replaces delimiters with `\0`). Not thread-safe. Cannot be used with string literals.

---

## Character Classification and Conversion

These functions are in `<ctype.h>`.

| Function | Description |
|---|---|
| `isalnum(c)` | Is alphanumeric? |
| `isalpha(c)` | Is alphabetic? |
| `isdigit(c)` | Is digit? |
| `isspace(c)` | Is whitespace? |
| `isupper(c)` | Is uppercase? |
| `islower(c)` | Is lowercase? |
| `toupper(c)` | Convert to uppercase |
| `tolower(c)` | Convert to lowercase |

```c
#include <ctype.h>

char c = 'a';
if (islower(c)) {
    c = toupper(c);    // 'A'
}
```

---

## Common Pitfalls

### 1. Buffer Overflow

```c
char dest[5];
strcpy(dest, "Hello World");    // Buffer overflow (dest is too small)
```

### 2. Forgetting Null Termination

```c
char dest[10];
strncpy(dest, "Hello", 10);
// dest may not be null-terminated if src has 10+ characters
```

**Fix:** Always ensure null termination after `strncpy`:

```c
dest[9] = '\0';
```

### 3. Using `strtok` with String Literals

```c
char *s = "Hello,World";
strtok(s, ",");    // Undefined behavior (modifies read-only memory)
```

### 4. Using `strcmp` for Case-Insensitive Comparison

```c
strcmp("Hello", "hello") != 0    // They are different
```

**Fix:** Convert both strings to the same case first.

### 5. Confusing `strcpy` and `strncpy`

```c
strncpy(dest, src, strlen(src) + 1);    // Safer, but need to handle length
```

### 6. Not Handling `NULL` Return Values

```c
char *pos = strchr(str, 'x');
if (pos) {
    // Use pos
} else {
    // Not found
}
```

---

## Complete Example

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

int main(void) {
    // Length
    char str[] = "Hello World";
    printf("strlen: %zu\n", strlen(str));
    
    // Copying
    char dest[20];
    strcpy(dest, str);
    printf("strcpy: %s\n", dest);
    
    // Concatenation
    char buffer[50] = "Hello";
    strcat(buffer, " World");
    printf("strcat: %s\n", buffer);
    
    // Comparison
    int cmp = strcmp("Hello", "World");
    printf("strcmp(\"Hello\", \"World\") = %d\n", cmp);
    
    // Searching
    char *pos = strchr(str, 'o');
    if (pos) {
        printf("strchr('o'): %s (position: %zu)\n", pos, pos - str);
    }
    
    char *sub = strstr(str, "World");
    if (sub) {
        printf("strstr(\"World\"): %s (position: %zu)\n", sub, sub - str);
    }
    
    // Tokenization
    char token_str[] = "apple,banana,grape,orange";
    char *token = strtok(token_str, ",");
    printf("strtok:\n");
    while (token != NULL) {
        printf("  %s\n", token);
        token = strtok(NULL, ",");
    }
    
    // Character classification
    char letters[] = "Hello123";
    for (int i = 0; letters[i]; i++) {
        if (isalpha(letters[i])) {
            printf("%c is a letter\n", letters[i]);
        } else if (isdigit(letters[i])) {
            printf("%c is a digit\n", letters[i]);
        }
    }
    
    // Case conversion
    char word[] = "Hello";
    for (int i = 0; word[i]; i++) {
        word[i] = tolower(word[i]);
    }
    printf("tolower: %s\n", word);
    
    // Safe formatting
    char safe[20];
    snprintf(safe, sizeof(safe), "Value: %d", 42);
    printf("snprintf: %s\n", safe);
    
    // strcspn example
    char input[] = "Hello123World";
    size_t num_len = strcspn(input, "0123456789");
    printf("strcspn: first digit at position %zu\n", num_len);
    
    return 0;
}
```

---

## Summary Table

| Function | Purpose | Return Value |
|---|---|---|
| `strlen(s)` | Length of string | `size_t` |
| `strcmp(s1, s2)` | Compare strings | 0 if equal |
| `strncmp(s1, s2, n)` | Compare up to n chars | 0 if equal |
| `strcpy(dest, src)` | Copy string | `dest` pointer |
| `strncpy(dest, src, n)` | Copy up to n chars | `dest` pointer |
| `strcat(dest, src)` | Concatenate strings | `dest` pointer |
| `strncat(dest, src, n)` | Concatenate up to n chars | `dest` pointer |
| `strchr(s, c)` | Find character | Pointer or `NULL` |
| `strrchr(s, c)` | Find character from end | Pointer or `NULL` |
| `strstr(haystack, needle)` | Find substring | Pointer or `NULL` |
| `strtok(str, delim)` | Tokenize string | Token or `NULL` |

---

## Cross-References

- **Previous:** [19 — Strings](19-strings.md)
- **Next:** [21 — Array and Pointer Relationship](21-array-pointer.md)
- **Character classification:** `<ctype.h>`
- **Memory manipulation:** `<string.h>` also includes `memcpy`, `memmove`, `memset`, `memcmp`

---

## References

- ISO/IEC 9899:2018 §7.24 — String handling `<string.h>`
- ISO/IEC 9899:2018 §7.4 — Character handling `<ctype.h>`
- `man string` — String handling functions