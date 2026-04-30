# C++ Module 08 - Containers, Iterators, Templates

This repository contains three exercises from C++ Module 08:
- `ex00`: generic search with `easyfind`
- `ex01`: range and span computation with `Span`
- `ex02`: iterable stack adapter with `MutantStack`

All exercises compile with:
- `c++`
- flags: `-Wall -Wextra -Werror -std=c++98`

## Repository Layout

```text
cpp_08/
├── ex00/
│   ├── easyfind.hpp
│   ├── easyfind.tpp
│   ├── main.cpp
│   └── makefile
├── ex01/
│   ├── Span.hpp
│   ├── Span.cpp
│   ├── main.cpp
│   └── makefile
└── ex02/
    ├── MutantStack.hpp
    ├── MutantStack.tpp
    ├── main.cpp
    └── makefile
```

## Build and Run

From each exercise directory:

```bash
make
./easyfind     # in ex00
./span         # in ex01
./mutantStack  # in ex02
```

Clean targets:

```bash
make clean
make fclean
make re
```

## Exercise Summary

### ex00 - `easyfind`
- Implements a template function to locate an `int` inside a generic STL container.
- Uses `std::find(begin, end, value)`.
- Throws `ValueNotFoundException` when the value does not exist.
- Supports both mutable and const containers via overloads.

### ex01 - `Span`
- Stores up to `N` integers.
- `addNumber(int)`: inserts one value, throws if full.
- `addNumbers(begin, end)`: bulk insert from iterator range, throws if capacity is exceeded.
- `shortestSpan()`: sorts a copy and computes minimum adjacent distance.
- `longestSpan()`: computes `max - min`.
- Throws `NotEnoughNumbersException` when fewer than two elements are stored.

### ex02 - `MutantStack`
- Inherits from `std::stack<T>`.
- Exposes iterators from the underlying container (`this->c`) so stack data can be traversed.
- Provides:
  - `begin()/end()`
  - `rbegin()/rend()`
  - const and non-const variants
- Keeps normal stack behavior (`push`, `pop`, `top`, `size`) and allows STL algorithm usage.

## ASCII Flow (Detailed)

```text
+------------------+
| Start Repository |
+------------------+
          |
          v
+---------------------------+
| Choose Exercise Directory |
| ex00 / ex01 / ex02        |
+---------------------------+
          |
          v
+-------------------+
| Run `make`        |
+-------------------+
          |
          v
+-------------------------------+
| Execute Binary                |
| ex00: ./easyfind              |
| ex01: ./span                  |
| ex02: ./mutantStack           |
+-------------------------------+
          |
          v
+------------------------------------------------+
| Runtime Logic                                  |
|                                                |
| ex00: easyfind(container, value)               |
|   -> std::find                                 |
|   -> found? yes: return iterator               |
|             no : throw ValueNotFoundException  |
|                                                |
| ex01: Span operations                          |
|   -> addNumber/addNumbers                      |
|   -> check capacity                            |
|   -> shortestSpan: sort copy + min diff        |
|   -> longestSpan : max - min                   |
|   -> invalid state? throw exceptions           |
|                                                |
| ex02: MutantStack operations                   |
|   -> push/pop/top/size from std::stack         |
|   -> iterate with begin/end/rbegin/rend        |
|   -> apply STL algorithms over iterators       |
+------------------------------------------------+
          |
          v
+--------------------+
| Print Test Results |
+--------------------+
          |
          v
+----------------+
| End Execution  |
+----------------+
```

## ASCII Flow (Compact)

```text
[Select exXX] -> [make] -> [run binary] -> [container logic] -> [output]

ex00: find int in container -> iterator / exception
ex01: store N ints -> shortest/longest span
ex02: stack + iterators -> traversal + STL algorithms
```

## Notes

- `ex01` shortest-span complexity is dominated by sorting: `O(n log n)`.
- `ex01` longest-span complexity is `O(n)`.
- `ex00` complexity depends on linear scan: `O(n)`.
- `ex02` iterator traversal complexity is linear in container size.
