# push_swap — Developer Documentation

This document explains the internal design and execution flow of this `push_swap` implementation, including bonus validation context.

## 1. High-level architecture

### 1.1 Core data structure

All stack logic is based on the doubly linked list node `t_stack` (declared in `push_swap.h`):

- `nbr`: original integer value
- `index`: normalized rank after indexing (`0..n-1`)
- `next`, `prev`: doubly linked list pointers
- additional fields (`may_med`, `min_mv`, `cost`, `dest_node`) exist for extended strategies

### 1.2 Program modules

- **Input pipeline**
  - `process_arguments.c`
  - `check_args.c`
  - `make_stack.c`
- **Indexing**
  - `index_numbers.c`
  - `ft_qsort.c`
- **Sorting strategies**
  - `small_sort.c`
  - `radix_sort.c`
  - `help_algoritm_funtions.c`
- **Primitive operations**
  - `push_a.c`, `push_b.c`, `swap.c`, `rotate.c`, `revrot.c`
- **Utilities**
  - `stk_lent.c`, `free_and_clean.c`, `error_sys.c`

## 2. End-to-end flow

From `main.c`:

1. Parse/validate args with `process_arguments(argc, argv)`.
2. Build stack `a` and assign normalized indexes.
3. Select algorithm:
   - `len <= 5` → `small_sort`
   - `len > 5` → `radix_sort`
4. Post-pass check:
   - if still not sorted, call `rotate_a_to_min`.
5. Free all allocated memory.

If input is already sorted or invalid, execution stops early according to current validation behavior.

## 3. Input parsing and validation

### 3.1 Accepted formats

- Multiple CLI tokens (`./push_swap 4 2 1`)
- Single string split by spaces (`./push_swap "4 2 1"`)

`pre_args` decides which path to use; when `argc == 2`, it uses `ft_split`.

### 3.2 Validation rules

Implemented in `check_args.c` and `make_stack.c`:

- token must be numeric with optional leading sign (`is_digit_valid`)
- range must fit signed 32-bit (`ft_atol` checks overflow)
- duplicates are rejected (`check_duplicates`)
- empty input is rejected
- on error, prints `Error\n` to `stderr`

### 3.3 Stack creation

`make_stk` iterates through normalized argument tokens and appends nodes with `add_new_node_2_stk`.

## 4. Index normalization

Radix sorting is performed on `index`, not on raw values:

1. `fill_array` copies all `nbr` values to an array.
2. `ft_qsort` sorts the array ascending.
3. `assign_indexes` maps each list node value to its rank.

This ensures dense non-negative keys suitable for bitwise radix passes.

## 5. Sorting strategies

## 5.1 `small_sort` path (`len <= 5`)

Implemented in `small_sort.c`:

- `len == 2`: `sort_two`
- `len == 3`: `sort_three` hardcoded state transitions
- `len > 3 && len <= 5`:
  1. repeatedly move minimum to stack `b` (`push_min_to_b`)
  2. sort remaining 3 on `a`
  3. push back everything from `b` to `a`

## 5.2 Radix path (`len > 5`)

Implemented in `radix_sort.c` + `help_algoritm_funtions.c`:

For each bit `i` from `0` to `max_bits - 1`:

1. iterate all nodes currently in `a`:
   - if bit `i` of `index` is `0` → `pb`
   - else → `ra`
2. push all staged nodes back from `b` to `a` (`pa` loop)

Helper functions:

- `get_max_bits`: number of significant bits in max index
- `radix_push_to_b`: one partition pass
- `radix_push_back_to_a`: restore phase

A `chunk_sort` prototype path is present but not selected by `main.c` in current flow.

## 6. Primitive operations and output contract

Each operation mutates linked lists and writes the corresponding instruction to `stdout`:

- push: `pa`, `pb`
- swap: `sa`, `sb`, `ss`
- rotate: `ra`, `rb`, `rr`
- reverse rotate: `rra`, `rrb`, `rrr`

This output stream is the contract consumed by checker tools.

## 7. Memory management

- `free_stack`: frees one linked list
- `free_split`: frees `char **` split arrays
- `free_all`: centralized cleanup helper
- `free_and_exit`: used on validation failure paths

Memory ownership is straightforward: the main path builds stack nodes once, mutates pointers in-place, and frees all at exit.

## 8. Bonus context

## 8.1 What is included in this repo

- Precompiled checker binaries in root:
  - `checker`
  - `checker_linux`

## 8.2 What is not included

- No checker source file (`checker.c`) in this repository.
- Root `makefile` does not define a `bonus` target.

## 8.3 How bonus is validated here

Use checker pipeline:

```bash
ARG="3 2 1"
./push_swap $ARG | ./checker_linux $ARG
```

Expected result is `OK` when operation sequence is valid and sorting is correct.

External tester scripts under `tester/` and `push_swap_tester/` also expose bonus-related checks (`-b` flags or checker integration).

## 9. Complexity overview

Let `n` be number of input integers.

- Indexing:
  - quicksort array: average `O(n log n)`
  - index assignment scan: `O(n^2)` (nested search in current implementation)
- Small sort (`n <= 5`): effectively constant-time for project constraints
- Radix sort:
  - passes: `O(log n)` on normalized indexes
  - each pass processes `n` elements
  - total: `O(n log n)` operations

Auxiliary memory:

- `O(n)` for indexing array
- `O(n)` linked-list storage for stack nodes

## 10. Extension points

If you want to evolve this codebase:

- replace nested index assignment with hash/map or binary-search mapping after sort
- enable `chunk_sort` as alternative strategy based on input size thresholds
- add explicit root `bonus` target when checker source is available
- add CI scripts that run a fixed random seed benchmark and checker validation
