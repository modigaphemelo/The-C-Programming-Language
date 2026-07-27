# 19: Strings — Character Arrays, Null Terminator

---

## Overview

In C, a string is a sequence of characters terminated by a null character (`'\0'`). This is the fundamental model for text in C. Strings are stored in character arrays, and all standard string functions operate on this null-terminated model.

**Key characteristics:**

- Stored in `char` arrays
- Terminated by a null character (`\0`)
- The null terminator is required for standard library functions
- Strings can be modified (if not read-only)
- No built-in string type (just arrays of characters)

---

## Declaration and Initialization

### String Literals (Read-Only)

```c
char *s = "Hello";          // Points to a string literal (read-only)
const char *s = "Hello";    // Better: const indicates read-only
```

String literals are stored in read-only memory. Attempting to modify them is undefined behavior.

### Character Arrays (Modifiable)

```c
char s1[6] = "Hello";       // Size 6: {'H','e','l','l','o','\0'}
char s2[] = "Hello";        // Size inferred: 6
char s3[10] = "Hello";      // Size 10: "Hello\0\0\0\0\0"
```

### Explicit Initialization

```c
char s1[] = {'H', 'e', 'l', 'l', 'o', '\0'};    // String (with terminator)
char s2[] = {'H', 'e', 'l', 'l', 'o'};           // Not a string (no terminator)
```

**The null terminator is what makes it a string.** Without it, you just have an array of characters.

---

## The Null Terminator

The null terminator (`\0`, ASCII 0) marks the end of a string. It is not a printable character and cannot be entered from the keyboard. It is used internally by all C string functions to know where the string ends.

```c
char str[] = "Hello";
// Memory: H e l l o \0
// Index:  0 1 2 3 4 5
// Length: 5 (not including \0)
// Size:   6 (including \0)
```

**Functions like `strlen` count characters until they hit `\0`:**

```c
size_t len = strlen(str);    // 5
```

**Functions like `printf` print until they hit `\0`:**

```c
printf("%s", str);    // "Hello"
```

---

## Strings vs Character Arrays

| Code | Type | Terminator | Modifiable |
|---|---|---|---|
| `char *s = "Hello";` | pointer to literal | Yes | No |
| `char s[] = "Hello";` | array | Yes | Yes |
| `char s[6] = "Hello";` | array | Yes | Yes |
| `char s[] = {'H','e','l','l','o'};` | array | No | Yes |
| `char s[6] = "Hello";` | array | Yes | Yes |

**`char *s = "Hello"` vs `char s[] = "Hello"`:**

```c
char *p = "Hello";    // p points to a string literal in read-only memory
char a[] = "Hello";   // a is a modifiable array on the stack

p[0] = 'h';           // Undefined behavior (segmentation fault on many systems)
a[0] = 'h';           // OK, "hello"
```

**Recommendation:** Use `const char *` for read-only strings and `char []` for modifiable strings.

---

## String Length

```c
#include <string.h>

char str[] = "Hello";
size_t len = strlen(str);    // 5 (characters)
size_t size = sizeof(str);   // 6 (including \0)
```

**Your own version (without `strlen`):**

```c
size_t my_strlen(const char *s) {
    size_t n = 0;
    while (s[n] != '\0') {
        n++;
    }
    return n;
}
```

---

## Common String Functions

### `<string.h>` Functions

| Function | Description |
|---|---|
| `strlen(s)` | Length of string (excluding `\0`) |
| `strcpy(dest, src)` | Copy string (unsafe, no bounds check) |
| `strncpy(dest, src, n)` | Copy up to n characters (may not null-terminate) |
| `strcat(dest, src)` | Concatenate strings (unsafe) |
| `strncat(dest, src, n)` | Concatenate up to n characters |
| `strcmp(s1, s2)` | Compare strings (returns 0 if equal) |
| `strncmp(s1, s2, n)` | Compare up to n characters |
| `strchr(s, c)` | Find first occurrence of character |
| `strrchr(s, c)` | Find last occurrence of character |
| `strstr(haystack, needle)` | Find substring |
| `strtok(s, delim)` | Tokenize string (modifies input) |

### `strcpy` and `strncpy`

```c
char dest[10];
char src[] = "Hello";

strcpy(dest, src);     // dest = "Hello" (unsafe if dest is too small)
strncpy(dest, src, 9); // dest = "Hello\0\0\0\0" (safe)
dest[9] = '\0';        // Ensure null termination
```

**Warning:** `strncpy` does not guarantee null termination if the source is longer than the destination.

### `strcat` and `strncat`

```c
char dest[20] = "Hello";
char src[] = " World";

strcat(dest, src);           // dest = "Hello World"
strncat(dest, src, 5);       // dest = "Hello World" (only adds 5 chars)
dest[19] = '\0';             // Ensure null termination
```

### `strcmp` and `strncmp`

```c
int result;

result = strcmp("Hello", "Hello");   // 0 (equal)
result = strcmp("Hello", "World");   // Negative (H < W)
result = strcmp("Hello", "Hi");      // Negative (e < i)
result = strcmp("Hello", "Help");    // Negative (l < p)
result = strcmp("Hello", "Apple");   // Positive (H > A)
```

**Note:** `strcmp` compares lexicographically using ASCII values.

### `strtok` (Tokenization)

```c
char str[] = "Hello,World,Test";
char *token = strtok(str, ",");
while (token != NULL) {
    printf("%s\n", token);
    token = strtok(NULL, ",");
}
// Output: Hello, World, Test
```

**Warning:** `strtok` modifies the input string and is not thread-safe. It cannot be used with string literals.

---

## Reading and Writing Strings

### `printf` and `fprintf`

```c
printf("%s\n", str);            // Print to stdout
fprintf(stderr, "Error: %s\n", str);    // Print to stderr
```

### `puts` and `fputs`

```c
puts(str);           // Prints string with newline
fputs(str, stdout);  // Prints string without newline
```

### `scanf` and `fgets`

```c
char buffer[100];

// Unsafe (no bounds check)
scanf("%s", buffer);     // Danger: buffer overflow

// Safe
fgets(buffer, sizeof(buffer), stdin);    // Reads a line, including newline
```

**`fgets` reads a line including the newline character.** You may need to remove it:

```c
char *p = strchr(buffer, '\n');
if (p) {
    *p = '\0';    // Remove newline
}
```

### `sprintf` and `snprintf`

```c
char buffer[100];
int x = 42;
char *name = "Alice";

// Unsafe (no bounds check)
sprintf(buffer, "Name: %s, Age: %d", name, x);

// Safe
snprintf(buffer, sizeof(buffer), "Name: %s, Age: %d", name, x);
```

**Use `snprintf` for safe string formatting.**

---

## Common Pitfalls

### 1. Forgetting the Null Terminator

```c
char str[5] = "Hello";    // Error: no room for \0
char str[5] = {'H','e','l','l','o'};    // Not a string (no \0)
```

### 2. Buffer Overflow

```c
char buffer[10];
strcpy(buffer, "This string is too long");    // Buffer overflow
```

**Fix:** Use `strncpy`, `snprintf`, or `strlcpy` (non-standard).

### 3. Returning a Pointer to a Local String

```c
char *get_string(void) {
    char str[] = "Hello";
    return str;    // Dangerous: str is destroyed
}
```

### 4. Modifying a String Literal

```c
char *s = "Hello";
s[0] = 'h';    // Undefined behavior
```

### 5. Using `strtok` with a String Literal

```c
char *s = "Hello,World";    // String literal
strtok(s, ",");             // Undefined behavior (modifies read-only memory)
```

### 6. Confusing `sizeof` and `strlen`

```c
char str[] = "Hello";
size_t len = strlen(str);    // 5
size_t size = sizeof(str);   // 6
```

---

## Complete Example

```c
#include <stdio.h>
#include <string.h>
#include <ctype.h>

void remove_newline(char *s) {
    char *p = strchr(s, '\n');
    if (p) {
        *p = '\0';
    }
}

int main(void) {
    // Declaration
    char str1[20] = "Hello";
    char str2[] = "World";
    const char *str3 = "Hello, World!";
    
    // Length
    printf("str1: %s (len: %zu, size: %zu)\n", str1, strlen(str1), sizeof(str1));
    printf("str2: %s (len: %zu, size: %zu)\n", str2, strlen(str2), sizeof(str2));
    
    // Copy
    char dest[20];
    strcpy(dest, str1);
    printf("strcpy: %s\n", dest);
    
    // Concatenate
    strcat(dest, " ");
    strcat(dest, str2);
    printf("strcat: %s\n", dest);
    
    // Compare
    int cmp = strcmp(str1, str2);
    printf("strcmp(\"%s\", \"%s\") = %d\n", str1, str2, cmp);
    
    // Search
    char *pos = strstr(str3, "World");
    if (pos) {
        printf("Found 'World' at position: %zu\n", pos - str3);
    }
    
    // Tokenization
    char token_str[] = "Hello,World,Test,Tokens";
    printf("Tokens:\n");
    char *token = strtok(token_str, ",");
    while (token != NULL) {
        printf("  %s\n", token);
        token = strtok(NULL, ",");
    }
    
    // String formatting
    char buffer[100];
    int x = 42;
    snprintf(buffer, sizeof(buffer), "Value: %d", x);
    printf("snprintf: %s\n", buffer);
    
    // User input
    char input[100];
    printf("Enter a string: ");
    fgets(input, sizeof(input), stdin);
    remove_newline(input);
    printf("You entered: '%s'\n", input);
    
    // String conversion (character cases)
    char mixed[] = "Hello World!";
    printf("Original: %s\n", mixed);
    for (int i = 0; mixed[i]; i++) {
        mixed[i] = toupper(mixed[i]);
    }
    printf("Uppercase: %s\n", mixed);
    
    return 0;
}
```

---

## Cross-References

- **Previous:** [18 — Multidimensional Arrays](18-multidimensional.md)
- **Next:** [20 — String Functions](20-string-functions.md)
- **Pointers to characters:** [23 — Pointer Basics](/04-pointers/notes/23-pointer-basics.md)
- **Character arrays and functions:** [21 — Array and Pointer Relationship](21-array-pointer.md)

---

## References

- ISO/IEC 9899:2018 §7.24 — String handling `<string.h>`
- ISO/IEC 9899:2018 §5.2.1 — Character sets
- `man string` — String handling functions