# get_next_line

`get_next_line` is a 42 project that implements a function to read a file descriptor **line by line**.

This repository contains both:
- **Mandatory** version (`get_next_line.c`, `get_next_line_utils.c`)
- **Bonus** version (`get_next_line_bonus.c`, `get_next_line_utils_bonus.c`) with support for multiple file descriptors

---

## Function Prototype

```c
char *get_next_line(int fd);
```

### Behavior
- Returns the next line read from `fd`.
- Returned line includes `\n` if a newline was read.
- Returns `NULL` when EOF is reached or on read/allocation error.

---

## Project Structure

```text
get_next_line.c
get_next_line_utils.c
get_next_line.h

get_next_line_bonus.c
get_next_line_utils_bonus.c
get_next_line_bonus.h

gnlTester/
gnl-station-tester/
tester/
```

---

## Build

### Mandatory

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line.c get_next_line_utils.c
```

### Bonus

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=42 get_next_line_bonus.c get_next_line_utils_bonus.c
```

You can change `BUFFER_SIZE` at compile time:

```bash
cc -Wall -Wextra -Werror -D BUFFER_SIZE=1 get_next_line_bonus.c get_next_line_utils_bonus.c
```

---

## Usage Example

```c
int fd = open("file.txt", O_RDONLY);
char *line;

while ((line = get_next_line(fd)) != NULL)
{
    printf("%s", line);
    free(line);
}
close(fd);
```

---

## Bonus Details

The bonus implementation uses:

```c
static char *remainder[1024];
```

This stores an independent read state per file descriptor, allowing interleaved reads from multiple files/sockets.

---

## Testing

This repository already includes popular testers:
- `gnlTester/`
- `gnl-station-tester/`
- `tester/`

Example commands (from `gnlTester`):

```bash
cd gnlTester
make m   # mandatory
make b   # bonus
make a   # all
```

---

## Important Notes

- `get_next_line.c` and `get_next_line_bonus.c` currently include local `main` functions for manual testing.
- If you compile with external testers, remove/comment those `main` functions to avoid duplicate-symbol build errors.

For implementation internals, memory ownership, and known limitations, see `DevDoc.md`.
