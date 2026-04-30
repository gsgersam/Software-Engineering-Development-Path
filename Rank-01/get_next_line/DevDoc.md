# Developer Documentation (DevDoc)

## 1. Overview

This project implements `get_next_line(int fd)` for both mandatory and bonus requirements.

- Mandatory: one static remainder buffer.
- Bonus: one static remainder buffer per file descriptor.

Core idea:
1. Keep reading chunks of size `BUFFER_SIZE`.
2. Accumulate data in a persistent remainder string.
3. Extract one line to return.
4. Keep only the unread tail for the next call.

---

## 2. Files and Roles

### Mandatory
- `get_next_line.c`
  - Main algorithm and line extraction pipeline.
- `get_next_line_utils.c`
  - Utility functions (`ft_strlen`, `ft_calloc`, `ft_substr`, `ft_strjoin`, etc.).
- `get_next_line.h`
  - Public prototypes and `BUFFER_SIZE` macro.

### Bonus
- `get_next_line_bonus.c`
  - Same algorithm, adapted for per-fd static state.
- `get_next_line_utils_bonus.c`
  - Utility functions for bonus build.
- `get_next_line_bonus.h`
  - Bonus header.

---

## 3. Control Flow

## 3.1 `get_next_line`

High-level sequence:

1. Validate input (`fd`, `BUFFER_SIZE`).
2. Allocate temporary read `buffer` (`BUFFER_SIZE + 1`).
3. Loop until newline is found in remainder:
   - `read(fd, buffer, BUFFER_SIZE)`
   - Append chunk to remainder with `ft_strjoin`
4. Build output line from remainder (`ft_make_line`).
5. Trim consumed part from remainder (`ft_clean_buffer`).
6. Return line.

ASCII flow:

```text
+---------------------------+
| get_next_line(int fd)     |
+-------------+-------------+
        |
        v
   +---------------------+
   | Validate fd /       |
   | BUFFER_SIZE > 0     |
   +----------+----------+
        |
     invalid|yes
        v
      +-------+
      | NULL  |
      +-------+
        ^
        |
        no
        |
        v
   +---------------------+
   | Allocate buffer     |
   | (BUFFER_SIZE + 1)   |
   +----------+----------+
        |
    alloc fail|yes
        v
      +-------+
      | NULL  |
      +-------+
        ^
        |
        no
        |
        v
   +-------------------------------+
   | while no '\\n' in remainder      |
   +---------------+---------------+
           |
           v
      +--------------------------+
      | bytes = read(fd,buffer)  |
      +------------+-------------+
             |
      +------------+------------+
      |                         |
      v                         v
   +-------------+          +---------------------+
   | bytes <= 0  |  no ---> | remainder =         |
   | ?           |          | strjoin(remainder,  |
   +------+------+          | buffer)             |
      | yes             +----------+----------+
      v                            |
    +-------+                        |
    | break | <----------------------+
    +---+---+
      |
      v
   +---------------------+
   | free(buffer)        |
   +----------+----------+
        |
    bytes == -1 ?
      |yes      |no
      v         v
    +-------+   +--------------------------+
    | NULL  |   | line = ft_make_line(...) |
    +-------+   +------------+-------------+
                 |
                 v
          +--------------------------+
          | remainder =              |
          | ft_clean_buffer(...)     |
          +------------+-------------+
                 |
                 v
               +-----------+
               | return    |
               | line      |
               +-----------+
```

Mandatory stores state in:

```c
static char *remainder;
```

Bonus stores state in:

```c
static char *remainder[1024];
```

## 3.1.1 Bonus ASCII flow (`get_next_line_bonus`)

```text
+-----------------------------------+
| get_next_line_bonus(int fd)       |
+-------------------+---------------+
          |
          v
     +--------------------------+
     | Validate fd /            |
     | BUFFER_SIZE > 0 /        |
     | fd < 1024                |
     +------------+-------------+
            |
         invalid|yes
            v
          +-------+
          | NULL  |
          +-------+
            ^
            |
            no
            |
            v
     +--------------------------+
     | Allocate buffer          |
     +------------+-------------+
            |
        alloc fail|yes
            v
          +-------+
          | NULL  |
          +-------+
            ^
            |
            no
            |
            v
     +--------------------------------------+
     | while no '\\n' in remainder[fd]        |
     +-------------------+------------------+
               |
               v
         +--------------------------+
         | bytes = read(fd,buffer)  |
         +------------+-------------+
                |
          +---------+---------+
          |                   |
          v                   v
        +-----------+      +------------------------+
        |bytes <= 0?| no-> | remainder[fd] =        |
        +-----+-----+      | strjoin(remainder[fd], |
          | yes        | buffer)                |
          v            +-----------+------------+
        +-------+                    |
        | break | <------------------+
        +---+---+
          |
          v
      +------------------------+
      | free(buffer)           |
      +------------+-----------+
             |
        bytes == -1 ?
         |yes      |no
         v         v
      +---------------------+   +-----------------------------+
      | free(remainder[fd]) |   | line = ft_make_line(        |
      | remainder[fd]=NULL  |   |         remainder[fd])       |
      +----------+----------+   +--------------+--------------+
           |                             |
           v                             v
         +-------+              +--------------------------+
         | NULL  |              | remainder[fd] =          |
         +-------+              | ft_clean_buffer(...)     |
                    +------------+-------------+
                           |
                           v
                         +--------+
                         | return |
                         | line   |
                         +--------+
```

## 3.2 Helpers in `get_next_line*.c`

- `ft_new_line(char *s)`
  - Returns index of first `\n` or `-1` if not found.

- `ft_make_new_line(char *remainder, int index)`
  - Allocates and copies from start of remainder through newline.

- `ft_make_line(char *remainder)`
  - If newline exists, returns line up to newline.
  - Otherwise returns full remainder as final line.

- `ft_clean_buffer(char *remainder)`
  - Removes returned line from remainder.
  - Frees and returns `NULL` when no data is left.

---

## 4. Utility Layer

- `ft_strlen`: safe with `NULL` input (returns `0`).
- `ft_bzero` / `ft_calloc`: zero-initialized allocation with overflow check.
- `ft_substr`:
  - Allocates substring and frees original source pointer (ownership transfer pattern used by this code).
- `ft_strjoin`:
  - Allocates concatenated string.
  - In bonus utility implementation, it frees previous `s1` after copying.

---

## 5. Memory Ownership Model

- Returned line from `get_next_line` is heap-allocated.
  - **Caller must `free()` it.**

- Internal remainder is heap-allocated and persists across calls via `static`.

- `ft_clean_buffer` is responsible for turning consumed remainder into new smaller remainder.

- In this implementation style, some utility functions (notably `ft_substr` and bonus `ft_strjoin`) free input pointers. This is convenient but tightly couples call-sites to ownership rules.

---

## 6. Complexity

Let:
- `B = BUFFER_SIZE`
- `L = length of produced line`

Time per returned line is affected by repeated concatenation and copies, approximately linear to superlinear depending on number of joins.

Space:
- Temporary read buffer: `O(B)`
- Remainder: `O(unconsumed data)`
- Returned line: `O(L)`

Bonus adds up to 1024 independent remainders in worst case.

---

## 7. Mandatory vs Bonus

Only state storage changes:

- Mandatory: one shared static remainder.
- Bonus: indexed remainder (`remainder[fd]`) for interleaved FD reads.

Algorithm and helper behavior are otherwise almost identical.

---

## 8. Known Limitations in Current Code

1. Mandatory `ft_strjoin` currently returns `NULL` when `s1 == NULL`, which can break first append in `get_next_line`.
2. Bonus `get_next_line` does not check upper bound of `fd` against `1024` before indexing `remainder[fd]`.
3. On `read` error (`bytes_read == -1`), internal remainder for that FD is not explicitly cleared in current flow.
4. Source files include local `main` test functions; these conflict with external tester mains unless removed/commented.

---

## 9. Suggested Refactors

- Make `ft_strjoin` consistently `NULL`-safe in both mandatory and bonus.
- Add `fd >= 1024` guard in bonus.
- Ensure remainder cleanup on read error path.
- Remove test `main` from deliverable sources and keep test driver in separate files.
- Optionally reduce copy overhead with more efficient append strategy.

---

## 10. Maintenance Notes

- Keep function prototypes synchronized between `.h` and `.c` files.
- Validate with multiple `BUFFER_SIZE` values (e.g., `1`, `42`, very large).
- Test edge inputs:
  - Empty file
  - File with only `\n`
  - No trailing newline
  - Very long single line
  - Interleaved reads on multiple FDs (bonus)
