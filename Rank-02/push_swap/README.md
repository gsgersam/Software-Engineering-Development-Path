# push_swap

A 42-school `push_swap` implementation in C that sorts integers using two stacks (`a` and `b`) and prints the instruction sequence needed to sort stack `a` in ascending order.

This repository includes:
- the `push_swap` executable source code,
- Linux checker binaries (`checker`, `checker_linux`) for validation,
- extra tester suites under `tester/` and `push_swap_tester/`.

## Features

- Accepts both input formats:
  - multiple arguments: `./push_swap 3 2 1`
  - one quoted string: `./push_swap "3 2 1"`
- Validates input:
  - numeric tokens only (`+` / `-` allowed),
  - 32-bit signed integer range,
  - no duplicates,
  - error output as `Error` on `stderr`.
- Internal stack indexing (`0..n-1`) before sorting.
- Two sorting paths:
  - `small_sort` for small inputs (`<= 5`),
  - binary radix-based sorting for larger inputs (`> 5`).

## Build

From the project root:

```bash
make
```

This builds:
- `push_swap`
- `libft/libft.a`
- `ft_printf/libftprintf.a`

Useful targets:

```bash
make clean
make fclean
make re
```

## Usage

### Basic

```bash
./push_swap 2 1 3
./push_swap "2 1 3"
```

The program prints operations to `stdout` (for example `sa`, `pb`, `ra`, ...), one per line.

### Validate result with checker (Bonus workflow)

Because checker source is not part of this repository, validation is done with the provided Linux checker binaries.

If needed, grant execution permission:

```bash
chmod +x ./checker ./checker_linux
```

Then test a random case:

```bash
ARG="$(shuf -i 1-100 -n 100 | tr '\n' ' ')"
./push_swap $ARG | ./checker_linux $ARG
```

Expected output: `OK`.

## Bonus notes

- The repository includes precompiled checker binaries (`checker`, `checker_linux`) used to evaluate operation streams.
- There is no `checker` C source file here and no `make bonus` target in the root `makefile`.
- Bonus verification is still covered through:
  - checker execution pipeline,
  - tester scripts that support bonus mode.

## Testing

### Tester pack 1 (`tester/`)

```bash
cd tester
chmod +x push_swap_test_linux.sh
./push_swap_test_linux.sh
```

Bonus mode:

```bash
./push_swap_test_linux.sh -b
```

### Tester pack 2 (`push_swap_tester/`)

```bash
cd push_swap_tester
chmod +x basic_test.sh debug.sh loop.sh
./basic_test.sh
```

## Project layout (main files)

- `main.c`: entry point and strategy selection (`small_sort` vs radix).
- `process_arguments.c`, `check_args.c`, `make_stack.c`: parsing and validation.
- `index_numbers.c`, `ft_qsort.c`: index assignment over normalized value order.
- `small_sort.c`: dedicated path for very small stacks.
- `radix_sort.c`, `help_algoritm_funtions.c`: radix path and helpers.
- `push_a.c`, `push_b.c`, `swap.c`, `rotate.c`, `revrot.c`: stack operations.
- `free_and_clean.c`, `error_sys.c`: cleanup and error handling.

## Dependencies

- GCC/Clang with C compilation support
- `make`
- local submodules/libraries bundled in this repo:
  - `libft/`
  - `ft_printf/`

## License

No explicit license file is currently included in this repository.
