# libft

## Overview

libft is a custom C standard library created as part of the 42 curriculum. The goal of the project is to reimplement a selected set of libc functions and utility helpers while following strict coding rules, using only allowed system calls and standard headers.

This library provides:

- character classification and conversion functions
- memory manipulation functions
- string manipulation functions
- numeric conversion helpers
- file descriptor output helpers
- a singly linked list API as bonus functionality

The final result is a reusable static library: `libft.a`.

## Project goals

- understand how common C library functions work internally
- write safe and modular C code
- manage memory correctly
- build a static library that can be linked into other projects
- follow the 42 Norm and compilation constraints

## Build

### Requirements

- `cc` compiler
- `ar` archiver
- Unix-like environment

### Commands

- `make` builds `libft.a`
- `make bonus` builds the library including the linked list bonus functions
- `make clean` removes object files
- `make fclean` removes object files and the library
- `make re` rebuilds everything from scratch

## Included functions

### Part 1: character and memory functions

- `ft_isalpha`
- `ft_isdigit`
- `ft_isalnum`
- `ft_isascii`
- `ft_isprint`
- `ft_strlen`
- `ft_memset`
- `ft_bzero`
- `ft_memcpy`
- `ft_memmove`
- `ft_strlcpy`
- `ft_strlcat`
- `ft_toupper`
- `ft_tolower`
- `ft_strchr`
- `ft_strrchr`
- `ft_strncmp`
- `ft_memchr`
- `ft_memcmp`
- `ft_strnstr`
- `ft_atoi`

### Part 2: allocation and string helpers

- `ft_calloc`
- `ft_strdup`
- `ft_substr`
- `ft_strjoin`
- `ft_split`
- `ft_itoa`
- `ft_strmapi`
- `ft_striteri`
- `ft_putchar_fd`
- `ft_putstr_fd`
- `ft_putendl_fd`
- `ft_putnbr_fd`
- `ft_strtrim`

### Bonus: linked list utilities

- `ft_lstnew`
- `ft_lstadd_front`
- `ft_lstsize`
- `ft_lstlast`
- `ft_lstadd_back`
- `ft_lstdelone`
- `ft_lstclear`
- `ft_lstiter`
- `ft_lstmap`

## Linked list structure

The bonus part uses the following structure:

- `t_list` containing:
  - `void *content`
  - `struct s_list *next`

## Typical usage

Include the header in your project:

```c
#include "libft.h"
```

Compile and link the library with your program, then call the functions as needed.

## Notes

- All functions must respect the 42 constraints.
- Functions must handle edge cases carefully, especially when pointers are `NULL` or when sizes are zero.
- Bonus functions are only available when the library is built with `make bonus`.

## Directory contents

- source files for each function in the project root
- `libft.h` for prototypes and shared definitions
- `Makefile` for building the static library
- tester resources in `libft testers/`

## Author

Generated for the libft project.
