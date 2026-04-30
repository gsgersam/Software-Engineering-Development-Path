# CPP Module 06

42 C++ Module 06 exercises implemented in C++98.

This repository covers three topics:
- scalar conversion from string literals
- pointer serialization with `reinterpret_cast`
- runtime type identification with `dynamic_cast`

## Contents

- [Project Layout](#project-layout)
- [Requirements](#requirements)
- [Quick Start](#quick-start)
- [Exercises](#exercises)
- [ASCII Flow](#ascii-flow)
- [Common Make Targets](#common-make-targets)
- [Developer Notes](#developer-notes)

## Project Layout

```text
cpp_06/
├── ex00/
│   ├── ScalarConverter.hpp
│   ├── ScalarConverter.cpp
│   ├── main.cpp
│   └── makefile
├── ex01/
│   ├── Serializer.hpp
│   ├── Serializer.cpp
│   ├── main.cpp
│   └── makefile
├── ex02/
│   ├── Base.hpp
│   ├── Base.cpp
│   ├── A.hpp
│   ├── B.hpp
│   ├── C.hpp
│   ├── main.cpp
│   └── makefile
├── DevDoc.md
└── FLOW_ASCII.txt
```

## Requirements

- `c++` with C++98 support
- `make`
- Unix-like shell

Compile flags used in all exercises:

```text
-Wall -Wextra -Werror -std=c++98
```

## Quick Start

Build and run each exercise:

```bash
cd ex00 && make && ./convert 42
cd ../ex01 && make && ./serializer
cd ../ex02 && make && ./typeinfo
```

## Exercises

### ex00 — ScalarConverter (`convert`)

Build/run:

```bash
cd ex00
make
./convert <literal>
```

Examples:

```bash
./convert 42
./convert a
./convert 'z'
./convert 3.14
./convert 2.5f
./convert nan
./convert +inf
```

Summary:
- classifies input into scalar/pseudo-literal categories
- converts and prints as `char`, `int`, `float`, `double`
- handles range and displayability edge cases

### ex01 — Serializer (`serializer`)

Build/run:

```bash
cd ex01
make
./serializer
```

Summary:
- serializes `Data*` to `uintptr_t`
- deserializes `uintptr_t` back to `Data*`
- demonstrates pointer round-trip correctness

### ex02 — TypeInfo (`typeinfo`)

Build/run:

```bash
cd ex02
make
./typeinfo
```

Summary:
- randomly instantiates `A`, `B`, or `C` through `Base*`
- identifies runtime type with pointer/reference overloads
- validates RTTI behavior via repeated runs

## ASCII Flow

See [FLOW_ASCII.txt](FLOW_ASCII.txt) for the full text diagram.

```text
Start -> pick ex00/ex01/ex02 -> make -> run executable -> inspect casting behavior -> End
```

## Common Make Targets

Use inside any exercise directory:

```bash
make
make clean
make fclean
make re
```

## Developer Notes

- technical details are documented in [DevDoc.md](DevDoc.md)
- implementation keeps each exercise isolated and buildable independently
