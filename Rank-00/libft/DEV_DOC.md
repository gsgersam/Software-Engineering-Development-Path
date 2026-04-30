# libft Developer Documentation

## 1. Project purpose

`libft` is a foundational C library that reimplements a subset of the standard C library and adds a small set of utility helpers. It is designed to be used as a reusable static archive in later 42 projects.

The codebase is organized around three main areas:

1. character and memory primitives
2. string and conversion helpers
3. bonus linked list utilities

## 2. Build system

### Targets

- `all`: builds `libft.a`
- `bonus`: builds `libft.a` with linked list functions included
- `clean`: removes object files
- `fclean`: removes object files and `libft.a`
- `re`: full rebuild

### Output

The final archive is `libft.a`.

### Compiler settings

The project uses:

- `cc`
- `-Wall`
- `-Wextra`
- `-Werror`

This means warnings are treated as errors, so all code must compile cleanly.

## 3. Public interface

The public API is declared in `libft.h`.

### Shared types

- `t_list`: singly linked list node type used by bonus functions

### Mandatory functions

#### Character checks

- `ft_isalpha`
- `ft_isdigit`
- `ft_isalnum`
- `ft_isascii`
- `ft_isprint`

#### String length and search

- `ft_strlen`
- `ft_strchr`
- `ft_strrchr`
- `ft_strncmp`
- `ft_strnstr`

#### Memory

- `ft_memset`
- `ft_bzero`
- `ft_memcpy`
- `ft_memmove`
- `ft_memchr`
- `ft_memcmp`

#### String and conversion

- `ft_atoi`
- `ft_toupper`
- `ft_tolower`
- `ft_strdup`
- `ft_strlcpy`
- `ft_strlcat`
- `ft_calloc`
- `ft_substr`
- `ft_strjoin`
- `ft_split`
- `ft_itoa`
- `ft_strmapi`
- `ft_striteri`
- `ft_strtrim`

#### File descriptor output

- `ft_putchar_fd`
- `ft_putstr_fd`
- `ft_putendl_fd`
- `ft_putnbr_fd`

### Bonus linked list functions

- `ft_lstnew`
- `ft_lstadd_front`
- `ft_lstsize`
- `ft_lstlast`
- `ft_lstadd_back`
- `ft_lstdelone`
- `ft_lstclear`
- `ft_lstiter`
- `ft_lstmap`

## 4. Function grouping and responsibilities

### 4.1 Character functions

These functions are thin wrappers around common character classification and conversion logic.

Expected behavior:

- return a non-zero value for true conditions in checkers
- preserve the input character value semantics
- avoid side effects

### 4.2 Memory functions

These functions operate on raw memory blocks rather than C strings.

Important behaviors:

- `ft_memcpy` assumes non-overlapping regions
- `ft_memmove` must support overlap safely
- `ft_bzero` must clear exactly `n` bytes
- `ft_memset` must fill byte by byte using the provided value converted to `unsigned char`

### 4.3 String functions

These functions operate on null-terminated strings.

Important behaviors:

- guard against `NULL` inputs where appropriate
- return properly terminated strings for copy and join helpers
- handle empty strings and zero-length operations correctly

### 4.4 Allocation helpers

These functions may allocate memory and therefore need careful failure handling.

- `ft_calloc` must return zero-initialized memory
- `ft_strdup`, `ft_substr`, `ft_strjoin`, `ft_split`, and `ft_itoa` must return dynamically allocated results
- allocation failures should propagate as `NULL`

### 4.5 Output helpers

These functions write to a file descriptor.

- they should use `write()` or equivalent low-level output
- they should not depend on `printf`
- they should work with standard file descriptors such as `1` and `2`

### 4.6 Bonus linked list API

The list functions provide basic singly linked list management.

Expected responsibilities:

- `ft_lstnew` creates a node
- `ft_lstadd_front` inserts at the head
- `ft_lstadd_back` inserts at the tail
- `ft_lstlast` returns the last node
- `ft_lstsize` counts nodes
- `ft_lstdelone` deletes one node and applies a content deleter
- `ft_lstclear` frees the entire list
- `ft_lstiter` applies a callback to each content pointer
- `ft_lstmap` creates a new mapped list and must clean up on failure

## 5. Memory management rules

### Allocation ownership

Any function returning dynamically allocated memory transfers ownership to the caller.

### Cleanup expectations

- every allocation path must be paired with a failure cleanup strategy
- `ft_lstmap` must delete partially created nodes if mapping fails
- `ft_split` must free all already allocated substrings if any allocation fails

### Common pitfalls

- forgetting to null-terminate copied strings
- leaking memory on partial failure
- using overlapping memory with `ft_memcpy`
- failing to preserve original content in `ft_memmove`

## 6. Suggested implementation order

A practical order for maintenance or review is:

1. character functions
2. memory functions
3. string length and search helpers
4. copy and join utilities
5. allocation helpers
6. file descriptor output helpers
7. bonus linked list functions

This order reduces dependency issues because later functions often rely on earlier primitives such as `ft_strlen`, `ft_memcpy`, or `ft_calloc`.

## 7. Testing strategy

### Mandatory tests

Verify:

- `NULL` handling where applicable
- empty strings
- zero sizes
- overlapping memory for `ft_memmove`
- boundary cases for `ft_strlcpy` and `ft_strlcat`
- sign and whitespace parsing for `ft_atoi`
- allocation failures

### Bonus tests

Verify:

- empty lists
- single-node lists
- list extension at head and tail
- deletion callbacks
- mapping failure cleanup

### Recommended validation

Use both custom tests and the provided tester resources in the workspace to compare behavior against expected libc semantics.

## 8. Code conventions

- keep each function in its own source file
- keep headers minimal and centralized in `libft.h`
- avoid unnecessary global state
- prefer small helper functions only when they improve clarity and do not break project rules
- keep the API stable for reuse in other projects

## 9. Project layout

- source files in the project root
- `libft.h` as the public header
- `Makefile` for build automation
- tester assets under `libft testers/`

## 10. Maintenance checklist

Before submitting or reusing the library, confirm that:

- the project builds with `make`
- bonus builds with `make bonus`
- `make clean` and `make fclean` work correctly
- all public prototypes match their implementations
- all functions behave correctly for edge cases
- no memory leaks remain in allocation-heavy functions

## 11. Summary

This project implements a reusable C utility library with mandatory libc-like functions and bonus linked list support. The code should remain small, consistent, and easy to link into future 42 projects.
