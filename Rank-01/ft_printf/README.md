# ft_printf

A custom implementation of `printf` for the 42 Common Core, shipped as a static library.

This repository currently documents the **mandatory implementation** (`ft_printf`).

---

## 1) Project Goal

Recreate a subset of the standard C `printf` behavior using:
- variadic arguments (`stdarg.h`),
- low-level output (`write`),
- recursive number conversion,
- return value as the number of printed characters.

---

## 2) Supported Conversions (Mandatory)

The mandatory `ft_printf` supports:
- `%c` character
- `%s` string
- `%p` pointer (`0x...` or `(nil)`)
- `%d` signed decimal integer
- `%i` signed decimal integer
- `%u` unsigned decimal integer
- `%x` lowercase hexadecimal
- `%X` uppercase hexadecimal
- `%%` literal percent sign

### Behavior notes
- If `%s` receives `NULL`, output is `(null)`.
- If `%p` receives `0`, output is `(nil)`.
- `INT_MIN` is handled explicitly in decimal conversion.
- Unknown format specifiers fall through and currently contribute `0` printed characters.

---

## 3) Build

### Mandatory
From project root:

```bash
make
```

Output:
- `libftprintf.a`

### Cleanup

```bash
make clean
make fclean
make re
```

## 4) Quick Integration in Another Project

### Header include
```c
#include "ft_printf.h"
```

### Link flags example
```bash
cc main.c -L. -lftprintf -I. -o app
```

### Simple usage
```c
ft_printf("Name: %s, score: %d\n", "Alice", 42);
```

---

## 5) Repository Structure (Main Parts)

- `ft_printf.c`
  - Main loop + dispatch to conversion handlers.
- `ft_printf_utils1.c`
  - `ft_putchar`, integer decimal, unsigned decimal, `%x/%X`, `%%`.
- `ft_printf_utils2.c`
  - `ft_putstr`, pointer formatting, helper hexadecimal printer for pointers.
- `ft_printf.h`
  - Public API and helper prototypes.
- `libft/`
  - Local libft dependency.
- `tester/`, `tripouille/`
  - External test resources and harnesses.

---

## 6) High-Level Flow (ASCII)

### Mandatory `ft_printf`

```text
+-----------------------------+
| call ft_printf(format, ...) |
+-------------+---------------+
              |
              v
     +------------------+
     | validate format? |
     +--------+---------+
              | no
              v
           return -1
              |
              | yes
              v
   +--------------------------+
   | init va_list, i=0,len=0  |
   +------------+-------------+
                |
                v
      +----------------------+
      | while format[i] != 0 |
      +----------+-----------+
                 |
       +---------+---------+
       | format[i] == '%' ?|
       +----+----------+---+
            |yes       |no
            v          v
+-------------------+  +-------------------+
| handle specifier  |  | print format[i]   |
| format[i+1]       |  | with ft_putchar   |
+---------+---------+  +---------+---------+
          |                      |
          v                      v
      len += printed chars   len += 1
          |
          v
    i skips specifier
          |
          v
      i++ in loop
          |
          v
   +--------------------+
   | end loop, va_end   |
   +---------+----------+
             |
             v
         return len
```

### Dispatch branch map

```text
specifier -> function
-------------------------------------------
'c'       -> ft_putchar
's'       -> ft_putstr
'p'       -> ft_put_pointer
'd'/'i'   -> ft_putnbr_size
'u'       -> ft_putnbr_unsigned
'x'/'X'   -> ft_print_hexa
'%'       -> ft_percen_sign
other     -> return 0
```

---

## 7) Return Value Contract

`ft_printf` returns:
- total number of characters printed on success,
- `-1` only when `format == NULL`.

All helpers return how many characters they wrote, enabling accumulation.

---

## 8) Testing

Available in repository:
- `tester/` (shell-driven external tester)
- `tripouille/` (popular 42 testing setup)

Suggested validation order:
1. Build mandatory: `make`
2. Compare outputs with system `printf` for each specifier and edge case
3. Run your own targeted tests first
4. Run external testers after baseline confidence

---

## 9) Example Output Parity

Use these quick checks to compare behavior and return values against the system `printf`.

| Case | Call | Expected stdout | Expected return |
|------|------|------------------|-----------------|
| Character | `ft_printf("%c", 'A')` | `A` | `1` |
| String | `ft_printf("%s", "Hello")` | `Hello` | `5` |
| Null string | `ft_printf("%s", NULL)` | `(null)` | `6` |
| Signed int | `ft_printf("%d", -42)` | `-42` | `3` |
| Unsigned int | `ft_printf("%u", 42u)` | `42` | `2` |
| Hex lower | `ft_printf("%x", 255)` | `ff` | `2` |
| Hex upper | `ft_printf("%X", 255)` | `FF` | `2` |
| Pointer null | `ft_printf("%p", NULL)` | `(nil)` | `5` |
| Percent sign | `ft_printf("%%")` | `%` | `1` |

---

## 10) Current Limitations

- Mandatory does **not** implement full libc `printf` behavior (no width/precision parsing in mandatory).
- No buffering layer: each character is written directly with `write`.

---

## 11) Authoring Notes

This project follows 42 style constraints:
- C static library output,
- strict warnings (`-Wall -Wextra -Werror`),
- dependency on local `libft`.

For detailed internal implementation notes and the full execution flow, see `DEVDOC.md`.
