# Security Analysis Report: Secret in The Shadows

## Overview
The provided C program is designed to read a secret from a file and allow users to input data to attempt matching the secret. While the program appears functional at first glance, a thorough analysis reveals several security vulnerabilities, with one particularly critical issue.

## Analysis Methodology
1. **Code Review**: Manual inspection of all functions and control flows
2. **Memory Management Analysis**: Examination of allocation, usage, and deallocation patterns
3. **Input Validation Assessment**: Evaluation of user input handling
4. **Boundary Condition Testing**: Consideration of edge cases in buffer operations

## Identified Issues

### 1. Critical Vulnerability: Buffer Overflow in `secure_alloc`
The most severe issue lies in the `secure_alloc` function:

```c
void *secure_alloc(size_t size) {
    if (size > 1024) {                    
        printf("Allocation too large.\n");
        return NULL;
    }
    void *ptr = malloc(size + 1);
    if (!ptr) {
        perror("malloc");
        exit(1);
    }
    ((char *)ptr)[size] = '\0';
    return ptr;
}
```

**Problem**: While the function checks that the requested size isn't too large (<= 1024), it fails to enforce a minimum size validation. When `size` is 0:
- `malloc(size + 1)` allocates 1 byte
- The assignment `((char *)ptr)[size] = '\0'` writes to `ptr[0]`, which is valid
- However, when this buffer is used in `get_input()`, the while loop can write up to `len` bytes, where `len` could be 0

**Exploitation Scenario**:
1. User enters length 0
2. `secure_alloc(0)` returns a 1-byte buffer
3. The reading loop in `get_input()` uses the original `len` (0) as boundary
4. If the user provides any input, it will overflow the buffer

### 2. Secondary Issues

#### a. Memory Leak in `load_secret`
The function doesn't check if `secret_buffer` is already allocated before calling `malloc`, potentially leaking memory.

#### b. No Initialization of `secret_buffer`
The global variable `secret_buffer` isn't initialized to NULL, which could lead to undefined behavior if used before first allocation.

#### c. Potential Integer Overflow
In `secure_alloc`, `size + 1` could theoretically overflow if `size` is SIZE_MAX, though the size check makes this unlikely in practice.

## Impact Assessment
The buffer overflow vulnerability is particularly dangerous because:
1. It allows writing beyond allocated memory boundaries
2. Could potentially lead to arbitrary code execution
3. Might enable reading or overwriting the secret data
4. Could crash the program or lead to unpredictable behavior

## Proof of Concept
To trigger the vulnerability:
```
1. load secret
3. get input
Length: 0
Input: AAAAAAAAA... (many characters)
```

This would overflow the 1-byte buffer allocated for length 0.

## Recommendations

1. **Fix `secure_alloc`**:
```c
void *secure_alloc(size_t size) {
    if (size == 0 || size > 1024) {  // Add minimum size check
        printf("Invalid allocation size.\n");
        return NULL;
    }
    void *ptr = malloc(size + 1);
    if (!ptr) {
        perror("malloc");
        exit(1);
    }
    ((char *)ptr)[size] = '\0';
    return ptr;
}
```

2. **Improve `load_secret`**:
```c
void load_secret(){
    if (secret_buffer) {
        free(secret_buffer);
    }
    FILE *file = fopen("secret.txt", "r");
    if (!file) {
        perror("fopen");
        return;
    }
    secret_buffer = malloc(32);
    if (!secret_buffer) {
        perror("malloc");
        fclose(file);
        return;
    }
    fgets(secret_buffer, 32, file);
    fclose(file);
}
```

3. **Initialize `secret_buffer`**:
```c
char *secret_buffer = NULL;
```

