# pipex

A small Unix pipeline recreation in C.

This project reproduces the shell behavior of:

```bash
< infile cmd1 | cmd2 > outfile
```

using low-level system calls (`pipe`, `fork`, `dup2`, `execve`, `waitpid`).

## Features

- Creates a pipe between two commands.
- Reads from an input file and writes to an output file.
- Resolves command paths using the `PATH` environment variable.
- Uses a custom `libft` library.

## Build

From the project root:

```bash
make
```

This builds:

- `pipex`
- `libft/libft.a` (through the sub-make)

Useful targets:

```bash
make clean
make fclean
make re
```

## Usage

```bash
./pipex infile "cmd1 arg1 arg2" "cmd2 arg1 arg2" outfile
```

Example:

```bash
./pipex infile "grep hello" "wc -l" outfile
```

Equivalent shell command:

```bash
< infile grep hello | wc -l > outfile
```

## Program flow (high level)

1. Validate argument count (`argc == 5`).
2. Create a pipe.
3. `fork()` into child and parent.
4. Child process:
   - opens `infile`
   - redirects `stdin` to `infile`
   - redirects `stdout` to pipe write-end
   - executes `cmd1`
5. Parent process:
   - waits for child
   - opens/creates `outfile`
   - redirects `stdin` to pipe read-end
   - redirects `stdout` to `outfile`
   - executes `cmd2`

## Errors

On failure, the project exits with `EXIT_FAILURE` and prints a `perror` message.

Typical failures:

- wrong argument count
- pipe/fork failure
- file open failure
- command parsing/path resolution failure
- `execve` failure

## Project structure

- `pipex.c`: entry point + process and redirection logic
- `ft_tools.c`: error handling, PATH parsing, command execution helpers
- `pipex.h`: declarations and includes
- `libft/`: custom C standard library used by the project

## Notes

- This implementation handles exactly two commands (mandatory `pipex` behavior).
- Command parsing is based on splitting by spaces, so advanced shell quoting is not interpreted like a full shell.

## Quick test commands

Build and prepare input:

```bash
make
printf "hello\nworld\nhello 42\n" > infile
```

Happy path test:

```bash
./pipex infile "grep hello" "wc -l" outfile
cat outfile
```

Expected output:

```text
2
```

Compare against shell behavior:

```bash
< infile grep hello | wc -l
```

Missing infile test (expected failure):

```bash
./pipex missing.txt "cat" "wc -l" outfile
```

Unknown command test (expected failure):

```bash
./pipex infile "not_a_real_command" "wc -l" outfile
```
