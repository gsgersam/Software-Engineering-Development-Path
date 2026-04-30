# Developer Documentation (DevDoc)

This document describes design decisions, internal flow, and implementation details for each exercise in this repository.

## 1) ex00 - BitcoinExchange

### Purpose
Process a file of `date | value` entries and compute `value * bitcoin_rate` using historical rates from `data.csv`.

### Key Files
- `ex00/main.cpp`
- `ex00/BitcoinExchange.hpp`
- `ex00/BitcoinExchange.cpp`

### Core Data Structure
- `std::map<std::string, double> _data`
  - Key: date in `YYYY-MM-DD`
  - Value: exchange rate

### Main Execution Path
1. `main` validates argument count (`argc == 2`)
2. `loadDatabase("data.csv")`
3. `processInput(argv[1])`

### Validation Rules
- Date format must be `YYYY-MM-DD`
- Date must be calendar-valid (including leap years)
- Value must be:
  - positive (`>= 0`)
  - not larger than `1000`

### Rate Lookup Strategy
- Uses `lower_bound(date)` on the map:
  - exact match if present
  - otherwise nearest previous date

### Error Handling (Current behavior)
- File open failures produce explicit error messages
- Bad input lines print `Error: bad input => ...`
- Negative / too large values print dedicated messages

---

## 2) ex01 - RPN

### Purpose
Evaluate an arithmetic expression in Reverse Polish Notation.

### Key Files
- `ex01/main.cpp`
- `ex01/RPN.hpp`
- `ex01/RPN.cpp`

### Core Data Structure
- `std::stack<int> _stack`

### Supported Tokens
- Operands: single digit `0..9`
- Operators: `+`, `-`, `*`, `/`
- Spaces are used as token separators

### Evaluation Steps
1. Reset stack
2. Scan expression left to right
3. Push digits
4. On operator, pop two values, compute, push result
5. At end, stack size must be exactly 1

### Error Cases
- Empty expression
- Multi-digit token without separator
- Unknown character
- Missing operands
- Division by zero
- Final stack size different from 1

All error branches print `Error`.

---

## 3) ex02 - PmergeMe

### Purpose
Sort a sequence using a Ford-Johnson style merge-insertion approach and compare timings for `std::vector` and `std::deque`.

### Key Files
- `ex02/main.cpp`
- `ex02/PmergeMe.hpp`
- `ex02/PmergeMe.cpp`

### Input Rules
- Accepts only positive numeric CLI tokens
- Invalid tokens are rejected before processing

### Internal State
- `_vector` and `_deque` store identical input values
- `_timeVector` and `_timeDeque` store elapsed microseconds

### Algorithm Outline
1. Build ordered pairs (`a`, `b`) and normalize each pair so `a >= b`
2. Store larger values in `mainChain`, smaller values in `pend`
3. If odd count, append the leftover element to `mainChain`
4. Recursively call Ford-Johnson on `mainChain` until base case (`size <= 1`)
5. During recursion unwind, for each level:
  - compute Jacobsthal-driven index order over that level's `pend`
  - insert each pending value into that level's `mainChain` using `lower_bound`
6. Copy reconstructed `mainChain` back to the original container level

Same logic is implemented for both container types.

### Timing
- Uses `clock()` and converts elapsed ticks to microseconds
- Prints processing time per container

---

## ASCII Flows by Exercise (Full)

### ex00 - BitcoinExchange

```text
argv[1] -> open input file
      + load data.csv into map<date, rate>
         |
         v
      read each "date | value" line
         |
         v
      validate date + validate value
      |ok                 |fail
      v                   v
    lower_bound(date)      print row error
    (exact or previous)
      |
      v
    compute value * rate
      |
      v
    print output line
```

### ex01 - RPN

```text
argv[1] expression
  |
  v
reset stack -> scan chars/tokens
  |
  +-- digit ------> push
  |
  +-- operator ---> pop2, calc, push
  |
  +-- invalid ----> Error

end: stack size == 1 ?
     |yes        |no
     v           v
  print top    Error
```

### ex02 - PmergeMe (Ford-Johnson recursion detail)

```text
Input numbers -> duplicated in vector/deque
         |
         v
    fordJohnsonSort(C)
         |
         v
      if |C| <= 1: return
         |
         v
  build normalized pairs (a >= b)
  mainChain <- all a
  pend      <- all b
  odd leftover -> mainChain
         |
         v
  recurse: fordJohnsonSort(mainChain)
  (subdivide repeatedly until size 1)
         |
         v
  unwind this level:
   - jacob = Jacobsthal(pend.size())
   - for idx in jacob: binary insert pend[idx]
     into current mainChain (lower_bound)
         |
         v
  copy mainChain back to C
  (repeat unwind for upper levels)
         |
         v
     fully sorted C + measured time
```

## ASCII Flows by Exercise (Compact)

```text
ex00: input rows -> validate -> map lower_bound(date) -> conversion/error
ex01: tokens -> stack machine (+,-,*,/) -> single result/Error
ex02: pair split -> recurse mainChain to base -> Jacobsthal insertion on each unwind level -> sorted+timings
```

---

## Build & Run Reference

```bash
# ex00
cd ex00 && make && ./btc input.txt

# ex01
cd ../ex01 && make && ./RPN "7 7 * 7 -"

# ex02
cd ../ex02 && make && ./PmergeMe 9 3 5 1 8 2
```

## Quick Execution Examples

| Exercise | Minimal Command | Sample Input | Expected Shape of Output |
|---|---|---|---|
| `ex00` | `./btc input.txt` | `date | value` rows | `YYYY-MM-DD => value = converted_value` (or row-level error) |
| `ex01` | `./RPN "7 7 * 7 -"` | space-separated tokens | single integer result (or `Error`) |
| `ex02` | `./PmergeMe 9 3 5 1 8 2` | positive integer args | `Before`, `After`, vector/deque timing lines |

Example snippets:

```text
ex00 input row: 2011-01-03 | 3.5
ex01 expr:      8 9 * 9 -
ex02 args:      3 5 9 7 4
```

## Extension Ideas
- `ex00`: stricter numeric parsing feedback for malformed values
- `ex01`: support multi-digit and signed operands
- `ex02`: larger benchmark harness and averaged timing runs
