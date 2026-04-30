# pipex Developer Documentation

## 1. Scope

`pipex` implements a minimal two-command pipeline:

```bash
< infile cmd1 | cmd2 > outfile
```

The current codebase focuses on the mandatory part: a single pipe between two commands.

---

## 2. Build and dependencies

### Compiler and flags

- Compiler: `gcc`
- Flags: `-Wall -Wextra -Werror`

### External/internal dependency

- Internal static library: `libft/libft.a`
- Used helper functions include `ft_split`, `ft_strncmp`, `ft_strjoin`, etc.

### Build graph

1. Build object files from:
   - `pipex.c`
   - `ft_tools.c`
2. Build `libft.a` via `make -C libft`
3. Link final binary: `pipex`

---

## 3. Runtime contract

### CLI contract

```bash
./pipex infile "cmd1 ..." "cmd2 ..." outfile
```

Expected `argc` is `5`.

### Inputs and outputs

- Input file: `argv[1]`
- First command string: `argv[2]`
- Second command string: `argv[3]`
- Output file: `argv[4]`

---

## 4. Control flow architecture

Main logic is in `pipex.c`:

1. Validate argument count.
2. Create pipe (`int fd[2]`).
3. Fork.
4. Child branch (`ft_son`): configures infile + pipe write side, then executes command 1.
5. Parent branch (`ft_dad`): waits child, configures pipe read side + outfile, then executes command 2.

Important: both branches end in `execve` on success; they do not return to `main`.

### ASCII flow

```text
./pipex infile "cmd1" "cmd2" outfile
      |
      v
   +------------------+
   | argc == 5 ?      |
   +------------------+
     | yes         | no
     v             v
   +----------------+   error_exit("Invalid args")
   | pipe(fd)       |
   +----------------+
     |
     v
   +----------------+
   | pid = fork()   |
   +----------------+
      | child                 | parent
      v                       v
+-------------------+   +-------------------+
| ft_son(...)       |   | waitpid(pid, ...) |
| open(infile, R)   |   +-------------------+
| dup2(infile, 0)   |              |
| dup2(fd[1], 1)    |              v
| close(fd[0])      |   +-------------------+
| execute(cmd1)     |   | ft_dad(...)       |
+-------------------+   | open(outfile, W)  |
         | dup2(fd[0], 0)    |
         | dup2(outfile, 1)  |
         | close(fd[1])      |
         | execute(cmd2)     |
         +-------------------+

Data stream:
infile --> [cmd1] --> pipe(fd[1] -> fd[0]) --> [cmd2] --> outfile
```

---

## 5. Function-by-function reference

## `pipex.c`

### `int main(int argc, char **argv, char **envp)`

- Verifies `argc == 5`.
- Creates pipe with `pipe(fd)`.
- Calls `fork()`.
- Child process executes `ft_son`.
- Parent process waits for child with `waitpid`, then executes `ft_dad`.

### `void ft_son(char **argv, char **envp, int *fd)`

- Opens infile (`open(argv[1], O_RDONLY | O_RDWR)`).
- Redirects:
  - `stdin` <- infile
  - `stdout` <- pipe write end (`fd[1]`)
- Closes unused read end (`fd[0]`).
- Calls `execute(argv[2], envp)`.

### `void ft_dad(char **argv, char **envp, int *fd)`

- Opens/creates outfile (`open(argv[4], O_WRONLY | O_CREAT | O_TRUNC, 0644)`).
- Redirects:
  - `stdin` <- pipe read end (`fd[0]`)
  - `stdout` <- outfile
- Closes unused write end (`fd[1]`).
- Calls `execute(argv[3], envp)`.

## `ft_tools.c`

### `void error_exit(char *msg)`

- Wrapper around `perror(msg)` + `exit(EXIT_FAILURE)`.

### `char **get_paths(char **envp)`

- Finds `PATH=` entry in environment.
- Splits PATH by `:` and returns directory array.
- Fails if PATH is missing or split fails.

### `char *search_in_paths(char *cmd, char **paths)`

- Concatenates each PATH directory with `/` + command.
- Returns first candidate where `access(candidate, F_OK) == 0`.
- Returns `NULL` if not found.

### `char *find_path(char *cmd, char **envp)`

- If `cmd` exists directly (`access(cmd, F_OK)`), returns it.
- Otherwise resolves it through `PATH` using previous helpers.
- Exits on failure (`Command not found`).

### `void execute(char *cmd, char **envp)`

- Splits command string by spaces into `args`.
- Resolves executable path with `find_path(args[0], envp)`.
- Calls `execve(path, args, envp)`.
- Exits on parsing/path/exec failure.

---

## 6. File descriptor lifecycle

### Child

- Uses infile fd as source.
- Uses `fd[1]` (pipe write-end) as destination.
- Closes `fd[0]`.

### Parent

- Uses `fd[0]` (pipe read-end) as source.
- Uses outfile fd as destination.
- Closes `fd[1]`.

This mirrors the shell pipeline direction left-to-right.

---

## 7. Error handling strategy

All hard failures funnel through `error_exit`:

- immediate diagnostic via `perror`
- immediate process termination with non-zero status

There is no recovery mode; failures are fatal by design.

---

## 8. Known implementation limitations

1. **Command parsing is simplistic**: `ft_split(cmd, ' ')` does not fully support shell quoting/escaping.
2. **PATH search checks existence only**: `F_OK` is used instead of `X_OK`, so executability is not validated before `execve`.
3. **Resource cleanup is minimal**: early exits do not free all allocated intermediary strings/arrays.
4. **Infile open flags**: child opens infile with `O_RDONLY | O_RDWR`; read-only would be sufficient.
5. **Single pipeline only**: no support for bonus features like multiple commands or heredoc.

---

## 9. Extension points

If expanding this implementation:

- Add robust parser (quotes, escaped spaces, env expansion constraints as needed).
- Replace `F_OK` with `X_OK` during PATH resolution.
- Add centralized cleanup for allocated memory.
- Implement bonus mode (`here_doc`, N commands).
- Add structured test matrix for permission and missing-file edge cases.
