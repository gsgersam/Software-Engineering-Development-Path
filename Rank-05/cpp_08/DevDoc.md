# DevDoc - C++ Module 08

Technical reference for implementation details in `cpp_08`.

## 1) Development Targets

- Language standard: C++98
- Compiler expected: `c++`
- Warning policy: `-Wall -Wextra -Werror`
- Build system: per-exercise `makefile`

## 2) Module Architecture

```text
ex00 -> generic function template over STL containers (`easyfind`)
ex01 -> bounded integer container class (`Span`)
ex02 -> stack adapter exposing iterators (`MutantStack`)
```

## 3) ex00 - easyfind

### Public Interface

```cpp
template<typename T>
typename T::iterator easyfind(T& container, int value);

template<typename T>
typename T::const_iterator easyfind(const T& container, int value);
```

### Behavior
- Uses `std::find(container.begin(), container.end(), value)`.
- Returns iterator when a match exists.
- Throws `ValueNotFoundException` when no match is found.

### Error Contract
- `ValueNotFoundException::what()` returns: `"Value not found in container"`.

### Complexity
- Time: `O(n)`
- Extra space: `O(1)`

## 4) ex01 - Span

### Data Model
- `_n`: maximum allowed size.
- `_numbers`: `std::vector<int>` holding inserted values.

### Public API

```cpp
void addNumber(int number);
void addNumbers(std::vector<int>::iterator begin,
                std::vector<int>::iterator end);
unsigned int shortestSpan();
unsigned int longestSpan();
```

### Behavior Details
- `addNumber`: rejects insert when `_numbers.size() >= _n`.
- `addNumbers`: computes range size with `std::distance`; rejects if capacity would overflow.
- `shortestSpan`:
  1. Requires at least two values.
  2. Copies and sorts `_numbers`.
  3. Computes minimum adjacent absolute difference.
- `longestSpan`:
  1. Requires at least two values.
  2. Uses `std::min_element` and `std::max_element`.
  3. Returns `max - min`.

### Error Contract
- `ContainerFullException`: insertion exceeds fixed capacity.
- `NotEnoughNumbersException`: span queried with fewer than two values.

### Complexity
- `addNumber`: amortized `O(1)`
- `addNumbers`: `O(k)` insertion (plus distance computation)
- `shortestSpan`: `O(n log n)` (sort dominates)
- `longestSpan`: `O(n)`

## 5) ex02 - MutantStack

### Design
`MutantStack<T>` inherits from `std::stack<T>` and exposes iterators of the underlying protected container (`this->c`).

### Iterator Surface

```cpp
iterator begin();
iterator end();
const_iterator begin() const;
const_iterator end() const;
reverse_iterator rbegin();
reverse_iterator rend();
const_reverse_iterator rbegin() const;
const_reverse_iterator rend() const;
```

### Implementation Notes
- Keeps standard stack semantics intact.
- Adds compatibility with STL algorithms (`std::count` in test).
- Supports both mutable and const iteration paths.

### Complexity
- Stack operations keep the complexity of `std::stack`.
- Iteration is linear in the number of stored elements.

## 6) ASCII Flow (Detailed)

```text
+--------------------+
| Developer Workflow |
+--------------------+
          |
          v
+---------------------------------------------+
| Enter ex00 / ex01 / ex02                    |
| Build with `make`                           |
+---------------------------------------------+
          |
          v
+-------------------------------+
| Run generated executable      |
+-------------------------------+
          |
          v
+------------------------------------------------------+
| Exercise Runtime                                     |
|                                                      |
| ex00: easyfind                                       |
|   input: container + int                             |
|   process: std::find                                 |
|   output: iterator OR ValueNotFoundException         |
|                                                      |
| ex01: Span                                           |
|   input: numbers (single or range)                  |
|   process: capacity checks + storage                 |
|   query shortest: sort copy + adjacent min diff      |
|   query longest : min/max diff                       |
|   output: span value OR domain exceptions            |
|                                                      |
| ex02: MutantStack                                    |
|   input: stack operations (push/pop/top)             |
|   process: expose iterators from underlying container|
|   output: iterable stack + STL algorithm support     |
+------------------------------------------------------+
          |
          v
+-----------------------------+
| Console output / test logs  |
+-----------------------------+
```

## 7) ASCII Flow (Compact)

```text
[Build exXX] -> [Run binary] -> [Apply exercise logic] -> [Result/Exception]

ex00: search value
ex01: compute shortest/longest span
ex02: iterate stack content
```

## 8) Maintenance Tips

- Keep templates fully visible from headers (`.hpp` + included `.tpp`).
- Preserve exception guarantees when extending APIs.
- Prefer algorithm-based implementations over manual loops where possible.
- Keep tests in `main.cpp` focused on both success and failure paths.
