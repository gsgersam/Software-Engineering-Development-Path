# DEVDOC - minishell

Technical documentation for development, debugging, and maintenance.

## 1. Runtime Overview

The entry point is `src/binary_tree/main.c`.

- Initializes global state (`init_data`)
- Increments `SHLVL` (`bump_shlvl`)
- Selects mode:
  - Interactive when `isatty(0)` is true
  - Non-interactive when stdin is redirected
- On exit: frees environment list, clears readline history, and closes stdin/stdout backups

## 2. Core Data Structures

Defined in `inc/minishell.h`.

### `t_data`

Global execution context:

- Environment list (`env_list`)
- Current tokens (`tokens`)
- Current command tree (`root`)
- Input and prompt buffers
- Exit status (`exit_status`)
- Signal/interruption flags

### `t_node`

Execution tree node:

- `type`: node type (`CMD`, `PIPE`, `LOGICAL_AND`, etc.)
- `args`: command argv
- `redirs`: node redirections
- `writer` / `reader`: tree children

### `t_token`

Lexer unit:

- `type`
- `value`
- Quoting and spacing flags

### `t_env`

Linked-list environment variables (`key`, `value`, `next`).

## 3. Execution Pipeline

### Interactive Mode (`inter_mode.c`)

Main loop:

1. Build prompt (`built_prompt`)
2. Read input with `readline`
3. Handle `Ctrl-C`, EOF, and empty lines
4. Complete multiline input if quotes are unclosed
5. `tokenize`
6. `parse`
7. `expand_all_node_args`
8. `expand_wildcards_recursive`
9. `restore_masked_wildcards_recursive`
10. `prepare_heredocs`
11. `execute`
12. `free_tokens` and `free_tree`

### Non-Interactive Mode (`non_inter_mode.c`)

Equivalent flow, reading line-by-line using `get_next_line(0)`.

## 4. Lexer and Parser

### Tokenization

Location: `src/binary_tree/tokens/`

- Detects operators (`|`, `<`, `>`, `<<`, `>>`, `&&`, `||`, `;`, etc.)
- Respects quote states
- Ends token stream with `END`

### Parsing (Precedence)

Location: `src/binary_tree/parsing/bt_parser.c`

Hierarchy (lower to higher precedence):

1. `;`
2. `||`
3. `&&`
4. `|`
5. grouping / simple command / redirections

The parser builds a binary tree where operators are internal nodes and commands are leaf nodes (`CMD`).

## 5. Expansions

Applied before execution in this order:

1. Variable expansion in args
2. Wildcard expansion (`*`)
3. Restoration of quote-masked wildcards

Quoting semantics determine whether an expansion is allowed.

## 6. Bonus Feature: Wildcards

Wildcard support is treated as a bonus feature and is fully integrated into command processing.

Behavior and design:

- Pattern matching is handled through `match_pattern`.
- Node-level wildcard processing is handled by `expand_wildcards_node`.
- Full AST traversal is handled by `expand_wildcards_recursive`.
- Arguments are processed one by one through `process_single_argument`.
- Quote-masked wildcard symbols are restored by `restore_masked_wildcards_recursive`.

Practical notes:

- Wildcard expansion is applied after variable expansion.
- Quoted wildcard characters are protected from expansion.
- Expansion results are merged back into the command argument list before execution.

## 7. Heredoc

`prepare_heredocs` traverses the tree and processes each `HEREDOC` redirection before execution.

- Opens/creates a temporary fd for heredoc content
- Respects quoted delimiters (impacts internal expansion)
- Stores fd in the matching redirection (`node->redirs[i].fd`)

## 8. Execution

Main entry: `execute(t_data *d)` in `src/binary_tree/exec/exec_mgmt.c`.

Dispatch by node type:

- `CMD` -> builtin / external / redirection resolution
- `SUBSHELL` -> `execute_subshell`
- `PIPE` -> `execute_pipe`
- `SEMICOLON` -> `execute_semicolon`
- `LOGICAL_AND` / `LOGICAL_OR` -> `execute_logical`

### Builtins

Location: `src/builtins/`

- Parent-only: `cd`, `exit`, `unset`, `export`
- Forkable: `env`, `pwd`, `echo`

### External Commands

- Path lookup (`find_command_path`)
- `execve` with environment converted from linked list
- Return code `127` for command-not-found

## 9. Signals

Handlers in `src/binary_tree/utils/signal_handler.c`:

- `sigint_handler`
- `sigint_multiline_handler`
- `heredoc_sig_handler`

`setup_signals` and `reset_signals` are used depending on context (prompt, heredoc, execution).

## 10. Memory Management

Main cleanup routines:

- Tokens: `free_tokens`
- Tree: `free_tree`
- Environment: `free_env_list`

Practical rule: each command cycle must fully free temporary structures to avoid cumulative leaks.

## 11. Modules and Responsibilities

- `src/binary_tree/init/`: state and node bootstrap
- `src/binary_tree/tokens/`: lexer and wildcard logic
- `src/binary_tree/parsing/`: recursive parser and redirections
- `src/binary_tree/exec/`: dispatch and execution
- `src/binary_tree/utils/`: free logic, signals, utilities
- `src/builtins/`: builtins and environment operations

## 12. Recommended Debug Flow

- Print AST with `print_tree` / `debug_print_tree`
- Test by isolating phases: tokenize -> parse -> execute
- Compare against `bash` with `minitester/minitester.sh`
- Check leaks with:

```bash
make val
make val_child
make val_fds
```

## 13. Known Technical Risks

- Keep parser and executor behavior aligned when adding operators.
- Track string ownership carefully after expansions/token merges.
- Validate heredoc + signal + chained pipe corner cases.
