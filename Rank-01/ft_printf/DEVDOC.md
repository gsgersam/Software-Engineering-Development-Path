# DEVDOC — ft_printf (Mandatory)

This developer document describes the internal architecture and runtime behavior of the **mandatory** `ft_printf` implementation in this repository.

Scope of this document:
- mandatory path only (`ft_printf`, not bonus),
- function-level behavior,
- control flow and recursion model,
- edge-case behavior,
- complexity and extension notes.

---

## 1) Internal File Map

### Core entrypoint
- `ft_printf.c`
  - `ft_printf(const char *format, ...)`
  - internal dispatcher: `ft_handle_format(va_list args, const char format)`

### Primitive and numeric writers
- `ft_printf_utils1.c`
  - `ft_putchar`
  - `ft_print_hexa`
  - `ft_putnbr_size`
  - `ft_putnbr_unsigned`
  - `ft_percen_sign`

### String and pointer writers
- `ft_printf_utils2.c`
  - `ft_putstr`
  - `ft_putnbr_hexa`
  - `ft_put_pointer`

### Public interface
- `ft_printf.h`
  - exported prototype for `ft_printf`
  - helper function prototypes used across compilation units

---

## 2) API Contract

```c
int ft_printf(const char *format, ...);
```

### Input
- `format`: C string with plain characters and `%` conversion tokens.
- Variadic list: values consumed in order for each conversion specifier.

### Output
- Returns total number of characters written to file descriptor `1` (stdout).
- Returns `-1` only when `format == NULL`.

### Supported specifiers
- `c`, `s`, `p`, `d`, `i`, `u`, `x`, `X`, `%`

---

## 3) Runtime Execution Model

`ft_printf` uses a **single linear scan** over `format` and delegates conversion to specialized helpers.

### Main loop behavior
1. Validate `format` pointer.
2. Start `va_list`.
3. Iterate over each character in `format`.
4. If current char is `%`, dispatch according to next char.
5. Otherwise print literal char.
6. Accumulate printed character count from every helper.
7. End `va_list` and return accumulated count.

---

## 4) Full ASCII Flow (Mandatory)

```text
+------------------------------------------------------------------+
|                         ft_printf(format, ...)                   |
+-------------------------------+----------------------------------+
                                |
                                v
                      +-----------------------+
                      | format == NULL ?      |
                      +-----------+-----------+
                                  | yes
                                  v
                              return -1
                                  |
                                  | no
                                  v
                    +-------------------------------+
                    | va_start(args, format)        |
                    | i = 0, length = 0             |
                    +---------------+---------------+
                                    |
                                    v
                         +-----------------------+
                         | format[i] != '\0' ?   |
                         +-----+-----------------+
                               | no
                               v
                    +-------------------------------+
                    | va_end(args)                  |
                    | return length                 |
                    +-------------------------------+
                               ^
                               |
                               | yes
                               |
                +--------------+---------------+
                | format[i] == '%' ?           |
                +---------+--------------------+
                          | no
                          v
              +-------------------------------+
              | length += ft_putchar(format[i])|
              | i++                             |
              +------------------+-------------+
                                 |
                                 +---------------------> (loop)

                          yes
                           |
                           v
            +---------------------------------------------+
            | spec = format[i + 1]                        |
            | length += ft_handle_format(args, spec)      |
            | i++   // skip conversion character          |
            | i++   // loop increment                     |
            +------------------+--------------------------+
                               |
                               +-------------------------> (loop)
```

---

## 5) Dispatcher Logic (ft_handle_format)

Dispatch table:

```text
'c' -> ft_putchar(va_arg(args, int))
's' -> ft_putstr(va_arg(args, char *))
'p' -> ft_put_pointer(va_arg(args, unsigned long long))
'd' -> ft_putnbr_size(va_arg(args, int))
'i' -> ft_putnbr_size(va_arg(args, int))
'u' -> ft_putnbr_unsigned(va_arg(args, unsigned int))
'x' -> ft_print_hexa(va_arg(args, unsigned long long), 'x')
'X' -> ft_print_hexa(va_arg(args, unsigned long long), 'X')
'%' -> ft_percen_sign()
other -> 0
```

Important operational detail:
- Unknown specifiers are not printed literally by this implementation; they currently add `0` to output length.

---

## 6) Helper Function Semantics

### `ft_putchar(char c)`
- Calls `write(1, &c, 1)`.
- Returns `1`.

### `ft_putstr(char *s)`
- If `s == NULL`, writes `(null)` and returns `6`.
- Else writes characters one by one and returns string length.

### `ft_putnbr_size(int n)`
- Handles `INT_MIN` explicitly by printing `"-2147483648"`.
- For negative values, writes `'-'` and negates value.
- Uses recursion for decimal digit emission.
- Returns total printed length.

### `ft_putnbr_unsigned(unsigned long long n)`
- Prints `0` directly when `n == 0`.
- Otherwise recursive decimal emission.
- Returns printed digit count.

### `ft_print_hexa(unsigned int nbr, char format)`
- Selects alphabet: lowercase for `'x'`, uppercase for `'X'`.
- Recursively prints higher digits first (`nbr / 16`).
- Prints final digit (`nbr % 16`).
- Returns digit count.

### `ft_putnbr_hexa(unsigned long long nbr, char format)`
- Hex helper used by pointer formatting.
- Same recursive pattern as `ft_print_hexa`, but with wider input type.

### `ft_put_pointer(unsigned long long ptr)`
- If `ptr == 0`, prints `(nil)`.
- Else prints `0x` + lowercase hexadecimal address.
- Returns full printed length.

### `ft_percen_sign(void)`
- Writes `%`.
- Returns `1`.

---

## 7) Edge Cases and Expected Behavior

### Null format string
- Input: `ft_printf(NULL)`
- Output: nothing
- Return: `-1`

### Null string argument
- Input: `ft_printf("%s", NULL)`
- Output: `(null)`
- Return: `6`

### Null pointer
- Input: `ft_printf("%p", NULL)`
- Output: `(nil)`
- Return: `5`

### Signed integer minimum
- Input: `ft_printf("%d", INT_MIN)`
- Output: `-2147483648`
- Return: `11`

### Literal percent
- Input: `ft_printf("%%")`
- Output: `%`
- Return: `1`

---

## 8) Practical Validation Examples

These examples are useful to confirm both stdout content and character-count return value.

### Example A: mixed literal + specifiers
- Call: `ft_printf("User: %s | Score: %d\\n", "Ana", 42)`
- Expected stdout: `User: Ana | Score: 42` + newline
- Expected return: `22`

Count breakdown:
- `User: ` => 6
- `Ana` => 3
- ` | Score: ` => 10
- `42` => 2
- `\n` => 1
- Total => 22

### Example B: pointer + percent
- Call: `ft_printf("ptr=%p %% done", NULL)`
- Expected stdout: `ptr=(nil) % done`
- Expected return: `16`

Count breakdown:
- `ptr=` => 4
- `(nil)` => 5
- ` ` => 1
- `%` => 1
- ` done` => 5
- Total => 16

### Example C: signed integer boundaries
- Call: `ft_printf("%d %d", INT_MIN, INT_MAX)`
- Expected stdout: `-2147483648 2147483647`
- Expected return: `22` (11 + 1 + 10)

### Example D: unsupported specifier behavior
- Call: `ft_printf("A%qB")`
- Expected stdout with current implementation: `AB`
- Expected return with current implementation: `2`

Reason:
- `%q` falls into dispatcher default branch and contributes `0`.
- The scan still advances past `%` and `q`.

---

## 9) Complexity Notes

Let:
- `N` = length of format string,
- `K` = total printed characters for all converted values.

Time complexity is approximately:
- `O(N + K)`

Space complexity:
- `O(D)` recursion depth for number conversions,
- where `D` is number of digits in the current number (`base 10` or `base 16`).

No heap allocation is used by mandatory conversion helpers.

---

## 10) Design Choices

1. **Character-by-character writes**
   - Simpler logic and easy count tracking.
   - Trade-off: more syscalls than buffered output.

2. **Recursive number printing**
   - Compact and readable for digit order correctness.
   - Trade-off: small call-stack usage per conversion.

3. **Explicit special-case handling**
   - `INT_MIN`, `NULL` string, and `NULL` pointer are handled directly.

---

## 11) Known Mandatory Limitations

Compared to libc `printf`, this mandatory implementation intentionally does not include:
- width parsing,
- precision parsing,
- flag parsing (`-`, `0`, `+`, ` `, `#`),
- length modifiers (`hh`, `h`, `l`, `ll`, etc.),
- floating-point specifiers.

This is consistent with a standard 42 mandatory scope.

---

## 12) Suggested Debug Checklist (Mandatory)

When behavior differs from system `printf`:
1. Verify return value count (not only output text).
2. Validate `%p` null behavior (`(nil)`).
3. Validate `%s` null behavior (`(null)`).
4. Validate sign handling for `%d`/`%i` (`INT_MIN`, negatives, zero).
5. Validate `%x/%X` alphabet case.
6. Validate unsupported specifier behavior expected by your tests.

---

## 13) Minimal Call Graph (ASCII)

```text
ft_printf
 ├─ ft_putchar                  // literals
 └─ ft_handle_format
     ├─ ft_putchar              // %c
     ├─ ft_putstr               // %s
     ├─ ft_put_pointer          // %p
     │   └─ ft_putnbr_hexa
     ├─ ft_putnbr_size          // %d %i
     ├─ ft_putnbr_unsigned      // %u
     ├─ ft_print_hexa           // %x %X
     └─ ft_percen_sign          // %%
```

End of mandatory developer documentation.
