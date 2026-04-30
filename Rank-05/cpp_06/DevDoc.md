# DevDoc — CPP Module 06

Technical reference for the implementations in this repository.

## Table of Contents

- [Overview](#overview)
- [Build Profile](#build-profile)
- [ex00 — ScalarConverter](#ex00--scalarconverter)
- [ex01 — Serializer](#ex01--serializer)
- [ex02 — RTTI](#ex02--rtti)
- [Design Notes](#design-notes)

## Overview

The project is split into 3 independent exercises:

1. Parse and convert scalar literals (`ex00`)
2. Serialize/deserialize pointers (`ex01`)
3. Identify dynamic types via RTTI (`ex02`)

Each folder has its own `makefile` and binary.

## Build Profile

- Compiler: `c++`
- Standard: `C++98`
- Flags: `-Wall -Wextra -Werror -std=c++98`
- Make targets: `all`, `clean`, `fclean`, `re`

## ex00 — ScalarConverter

### Purpose

Convert one literal argument into:
- `char`
- `int`
- `float`
- `double`

### Key Interfaces

- `ScalarConverter::convert(const std::string&)`
- private helper: `getLiteralType(const std::string&)`

### Type Classification

Supported categories:
- pseudo-literals: `nan`, `nanf`, `+inf`, `+inff`, `-inf`, `-inff`, `inf`, `inff`
- scalar literals: `CHAR`, `INT`, `FLOAT`, `DOUBLE`
- invalid input: `ERROR`

### Implementation Notes

- Utility-style class (non-instantiable semantics)
- Uses `strtod`/`strtol` with `endptr` validation for strict parsing
- Uses stream extraction for conversion path after classification
- Shared print path applies range/displayability checks

### Behavior Notes

- Supports one-character input (`a`) and quoted char (`'a'`)
- Prints `impossible` for out-of-range conversions
- Prints `Non displayable` for non-printable character results

## ex01 — Serializer

### Purpose

Demonstrate reversible pointer serialization through an integer transport type.

### Data Model

```cpp
struct Data {
    int number;
    std::string name;
};
```

### Key Interfaces

- `static uintptr_t Serializer::serialize(Data* ptr)`
- `static Data* Serializer::deserialize(uintptr_t raw)`

### Implementation Notes

- Uses `reinterpret_cast` in both directions
- `main.cpp` validates address equality and data consistency after round-trip

## ex02 — RTTI

### Purpose

Identify the runtime type of derived instances (`A`, `B`, `C`) through `Base`.

### Class Structure

- `Base` is polymorphic (`virtual ~Base()`)
- `A`, `B`, `C` are empty public derivations from `Base`

### Key Interfaces

- `Base* generate()`
- `void identify(Base* p)`
- `void identify(Base& p)`

### Implementation Notes

- `generate()` chooses one derived type at random
- pointer overload uses `dynamic_cast<T*>` + null checks
- reference overload uses `dynamic_cast<T&>` + exception fallback
- `main.cpp` runs multiple iterations and frees allocated memory

## Design Notes

- Code is intentionally compact and exercise-focused
- Colored output is used for terminal readability but not required for logic
- All exercises are isolated to keep grading and testing straightforward

For high-level execution diagrams, see [FLOW_ASCII.txt](FLOW_ASCII.txt).
