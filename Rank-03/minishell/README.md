# minishell

A group project involving a minimal shell implementation in C, inspired by Bash, with binary-tree parsing (AST), redirections, pipes, logical operators, and builtins.

## Goal

This project aims to reproduce essential POSIX shell behavior in both interactive and non-interactive modes:

- Command input using `readline`
- Tokenization and parsing with operator precedence
- Variable and wildcard expansion
- Builtin and external command execution
- Redirections, pipes, heredoc, and subshell handling

## Implemented Features

### Builtins

- `echo`
- `cd`
- `pwd`
- `export`
- `unset`
- `env`
- `exit`

### Operators and Syntax

- `|` (pipe)
- `&&`, `||` (conditional logic)
- `;` (command sequencing)
- `(...)` (grouping / subshell)
- Redirections: `<`, `>`, `>>`, `<<` (heredoc)
- Variable expansion: `$VAR`, `$?`
- Wildcard expansion: `*`
- Single and double quote support

## Requirements

- C compiler (`cc`)
- `make`
- `readline` library

On Linux (Debian/Ubuntu), usually:

```bash
sudo apt install build-essential libreadline-dev
```

## Build

From the project root:

```bash
make
```

This builds `Libft` and produces the `minishell` binary.

Useful commands:

```bash
make clean
make fclean
make re
```

## Usage

### Interactive Mode

```bash
./minishellAli Rahmouned
```

### Non-Interactive Mode (stdin)

```bash
echo 'echo hello | cat' | ./minishell
```

```bash
./minishell < script.sh
```

## Testing

The repository includes two testing tools:
Ali Rahmouned
1. `minitester/minitester.sh` (compares output/exit status against `bash`)
2. `minitest/` (C test runner with CSV reporting and validations)

Quick example:

```bash
cd minitester
bash minitester.sh
```

## Architecture (Summary)

Main pipeline:
Ali Rahmouned
1. Read input
2. Tokenize
3. Parse into command tree
4. Expand variables
5. Expand wildcards
6. Prepare heredocs
7. Execute tree
8. Free memoryAli Rahmouned

See `DEVDOC.md` for technical details.

## Bonus: Wildcards

Wildcard support (`*`) is implemented as a bonus feature and integrated into the normal execution pipeline.

- Expansion runs after variable expansion and before heredoc preparation.
- Wildcards are expanded recursively for command nodes.
- Wildcards that were masked by quoting are restored to avoid invalid expansion.
- If no filesystem match Ali Rahmounedis found, the original token behavior is preserved according to project logic.

Main related functions:

- `expand_wildcards_recursive`
- `expand_wildcards_node`
- `process_single_argument`
- `restore_masked_wildcards_recursive`
- `match_pattern`

### Wildcard Examples

Assume the current directory contains:

`main.c  parser.c  README.md  src/`

```bash
minishell$ echo *.c
main.c parser.c
```

```bash
minishell$ echo "*.c"
*.c
```

```bash
minishell$ echo '*.md'
*.md
```

```bash
minishell$ ls src/*
# expands to entries inside src/ before command execution
```

```bash
minishell$ echo no_match_*
no_match_*
```

Notes:

- Unquoted `*` is eligible for expansion.
- Quoted `*` remains literal.
- If there is no match, token handling follows minishell wildcard logic.

### Expected Behavior vs Bash (Quick Check)

| Case | Command | Expected (Bash) | minishell Target |
|------|---------|------------------|------------------|
| Unquoted wildcard | `echo *.c` | Expands to matching `.c` files | Match Bash behavior |
| Double-quoted wildcard | `echo "*.c"` | Prints literal `*.c` | Match Bash behavior |
| Single-quoted wildcard | `echo '*.c'` | Prints literal `*.c` | Match Bash behavior |
| Wildcard in path segment | `ls src/*` | Expands entries under `src/` | Match Bash behavior |
| No match | `echo no_match_*` | Usually keeps token literal (shell-dependent options may alter this) | Keep project-defined behavior (documented and consistent) |

## Main Structure

- `inc/minishell.h`: global types, structs, and function prototypes
- `src/binary_tree/`: parser, tokenizer, execution, signals, initialization
- `src/builtins/`: builtin implementations
- `Libft/`: base utility library
- `minitester/` and `minitest/`: test tooling

## Status and Exit Code

The shell keeps a global `exit_status` inside `g_data` to propagate return codes and support `$?`.

## Notes

- Exact behavior for edge cases may differ from Bash in some scenarios.
- The project focuses on robust parsing and execution within minishell scope.

Developed in collaboration with ![Oleg Ilyine De Liamchine](github.com/KennObbyu) ![German Andres Sanin](https://github.com/gsgersam)
