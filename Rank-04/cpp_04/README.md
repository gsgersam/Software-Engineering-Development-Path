# cpp_04

C++ Module 04 focuses on **polymorphism**, **abstract classes**, **deep copies**, and **interfaces**.

## Exercises overview

### ex00 - Polymorphism
- Introduces a base `Animal` class and derived `Dog` / `Cat` classes.
- Demonstrates the difference between real polymorphism and a non-virtual design with `WrongAnimal` / `WrongCat`.

### ex01 - Deep copy and ownership
- Adds `Brain` to `Dog` and `Cat`.
- Focuses on the canonical form and deep copy behavior.
- Confirms that each object owns its own independent memory.

### ex02 - Abstract base class
- Replaces `Animal` with the abstract `AAnimal`.
- Prevents direct instantiation of the base class.
- Forces derived classes to implement the required behavior.

### ex03 - Interfaces and cloning
- Builds the Materia system with `AMateria`, `ICharacter`, and `IMateriaSource`.
- Uses `clone()` to create polymorphic copies.
- Manages inventory and runtime behavior through interfaces.

## Bonus emphasis

The **bonus concept** of this module is not only about passing the tests, but about showing clean object-oriented design:

- virtual destructors
- safe polymorphic deletion
- deep copy instead of shallow copy
- abstract classes and pure virtual methods
- interface-based architecture
- factory-style object creation with `clone()`

In short, the bonus is the proof that the design is correct, reusable, and memory-safe.

## Main project flow

```text
cpp_04
├── ex00
│   ├── Animal / Dog / Cat
│   └── WrongAnimal / WrongCat
├── ex01
│   ├── Animal / Dog / Cat
│   └── Brain for deep copy
├── ex02
│   └── AAnimal + abstract behavior
└── ex03
    ├── AMateria / Ice / Cure
    ├── Character
    └── MateriaSource
```

## Ex03 flow

```text
IMateriaSource
    |
    v
MateriaSource learns materia
    |
    v
createMateria(type)
    |
    v
clone() returns a new AMateria
    |
    v
Character equips materia
    |
    v
Character uses materia on target
```

## Build

Each exercise has its own Makefile.

```text
cd ex00 && make
cd ex01 && make
cd ex02 && make
cd ex03 && make
```

## Goal

The goal of this module is to understand how C++ handles polymorphism, memory ownership, and interface-driven design.
