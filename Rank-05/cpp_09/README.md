# CPP Module 09

This repository contains the three exercises of C++ Module 09:
- `ex00`: Bitcoin exchange rate evaluator
- `ex01`: Reverse Polish Notation (RPN) calculator
- `ex02`: Ford-Johnson based sorter and container timing (`std::vector` vs `std::deque`)

All exercises are built with:
- C++98
- flags: `-Wall -Wextra -Werror -std=c++98`

## Project Structure

```text
cpp_09/
├── ex00/
│   ├── BitcoinExchange.cpp
│   ├── BitcoinExchange.hpp
│   ├── data.csv
│   ├── input.txt
│   ├── main.cpp
│   └── makefile
├── ex01/
│   ├── RPN.cpp
│   ├── RPN.hpp
│   ├── main.cpp
│   └── makefile
└── ex02/
    ├── PmergeMe.cpp
    ├── PmergeMe.hpp
    ├── main.cpp
    └── makefile
```

## Build

Build each exercise inside its folder:

```bash
cd ex00 && make
cd ../ex01 && make
cd ../ex02 && make
```

Typical maintenance targets:

```bash
make clean
make fclean
make re
```

## Run

### ex00 - Bitcoin Exchange

```bash
cd ex00
./btc input.txt
```

Input file format:

```text
date | value
2009-01-03 | 2
2011-01-03 | 3.5
```

Output format:

```text
YYYY-MM-DD => value = converted_value
```

### ex01 - RPN

```bash
cd ex01
./RPN "8 9 * 9 - 9 - 9 - 4 - 1 +"
```

- Accepts single-digit operands
- Supports `+ - * /`
- Prints `Error` on invalid expression

### ex02 - PmergeMe

```bash
cd ex02
./PmergeMe 3 5 9 7 4
```

- Prints sequence before and after sorting
- Measures processing time for both `std::vector` and `std::deque`

## ASCII Flows by Exercise (Full)

### ex00 - BitcoinExchange

```text
+---------------------------+
| argv[1] input file path   |
+-------------+-------------+
                  |
                  v
+---------------------------+
| loadDatabase(data.csv)    |
| -> map<date, rate>        |
+-------------+-------------+
                  |
                  v
+---------------------------+
| for each input line       |
| "date | value"            |
+-------------+-------------+
                  |
                  v
+---------------------------+
| validate date + value     |
| (calendar + range checks) |
+------+------+-------------+
         |yes   |no
         |      v
         |   print Error
         v
+---------------------------+
| lower_bound(date) in map  |
| exact or closest previous |
+-------------+-------------+
                  |
                  v
+---------------------------+
| print conversion line     |
| date => value = result    |
+---------------------------+
```

### ex01 - RPN

```text
+---------------------------+
| argv[1] expression string |
+-------------+-------------+
                  |
                  v
+---------------------------+
| scan token by token       |
| (space-separated)         |
+------+------+-------------+
         |digit |operator
         |      |
         v      v
 push to   pop 2 values
 stack     compute, push
                  |
                  v
        invalid token /
        underflow / div0 ?
            |yes      |no
            v         v
        print Error  continue

end -> stack size == 1 ?
         |yes      |no
         v         v
    print top   print Error
```

### ex02 - PmergeMe (Ford-Johnson)

```text
+-------------------------------+
| argv numbers -> vector+deque  |
+---------------+---------------+
                     |
                     v
+-------------------------------+
| fordJohnsonSort(container)    |
+---------------+---------------+
                     |
                     v
        size <= 1 ?
         |yes      |no
         v         v
     return    make pairs
                  (a,b), order pair
                  mainChain <- larger
                  pend      <- smaller
                  odd tail -> mainChain
                     |
                     v
        recurse on mainChain
        until base case (1 elem)
                     |
                     v
        unwind each recursion level:
        build Jacobsthal order for pend
        insert pend[idx] with lower_bound
        into current mainChain
                     |
                     v
        copy mainChain back
        (level completed)
                     |
                     v
 final sorted container + timing
```

## ASCII Flows by Exercise (Compact)

```text
ex00: input file -> validate date/value -> lower_bound rate -> print conversion/error
ex01: expression -> token scan + stack ops -> final single value or Error
ex02: pair split -> recurse mainChain to base -> Jacobsthal-guided binary insert while unwinding -> sorted output + timings
```

## Quick I/O Examples

| Exercise | Command | Example Input | Example Output |
|---|---|---|---|
| `ex00` | `./btc input.txt` | `2011-01-03 | 3.5` | `2011-01-03 => 3.50 = ...` |
| `ex01` | `./RPN "8 9 * 9 -"` | `8 9 * 9 -` | `63` |
| `ex02` | `./PmergeMe 3 5 9 7 4` | `3 5 9 7 4` | `Before: ...` / `After: ...` + timing lines |

Notes:
- `ex00` prints one line per valid input row and specific errors for invalid rows.
- `ex01` prints `Error` if expression tokens/order are invalid.
- `ex02` prints sorted output for both containers and microsecond timings.

## Notes

- `ex00` uses `std::map<std::string, double>` and `lower_bound` for closest date lookup.
- `ex01` resets its internal stack on each evaluation.
- `ex02` uses the same algorithm on two containers to compare timings.

For implementation details, see [DevDoc.md](DevDoc.md).
