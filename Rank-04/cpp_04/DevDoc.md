# Developer Documentation - cpp_04

## Purpose

This module demonstrates core OOP concepts in C++98:

- polymorphism
- virtual functions and destructors
- abstract classes
- deep copy semantics
- interface-based design
- dynamic allocation and ownership

The code is split into four exercises, each one increasing the complexity of the design.

## Exercise breakdown

### ex00 - Runtime polymorphism

Goal:
- show how a base pointer can call derived behavior when methods are virtual
- contrast correct polymorphism with a non-virtual design

Key ideas:
- `Animal` is the polymorphic base
- `Dog` and `Cat` override the behavior
- `WrongAnimal` and `WrongCat` show what happens without virtual dispatch

Important detail:
- the base destructor must be virtual to delete derived objects safely through base pointers

### ex01 - Canonical form and deep copy

Goal:
- add a `Brain` member to `Dog` and `Cat`
- prove that copying creates independent internal state

Key ideas:
- copy constructor
- assignment operator
- destructor
- deep copy of dynamically allocated memory

Validation:
- changing the original object must not change the copied one

### ex02 - Abstract class

Goal:
- make the base animal abstract
- force derived classes to implement `makeSound()`

Key ideas:
- `AAnimal` cannot be instantiated
- pure virtual method
- base class still provides shared data and behavior

### ex03 - Interfaces, cloning, and inventory

Goal:
- build a materia system using interfaces
- create and use materias through polymorphism

Key ideas:
- `AMateria` is the abstract base class
- `Ice` and `Cure` are concrete materias
- `IMateriaSource` and `ICharacter` define the contract
- `MateriaSource` stores known materias and creates new ones with `clone()`
- `Character` equips, unequips, and uses materias

Memory model:
- `clone()` creates a new heap object
- `Character` owns equipped materias
- `MateriaSource` owns learned templates
- unequipped materias should be handled carefully to avoid leaks

## Bonus emphasis

The bonus value of this module is in the design quality:

- correct ownership rules
- safe polymorphic deletion
- copy safety for objects with heap members
- abstract interfaces instead of concrete coupling
- factory-like creation through cloning
- separation between learned templates and runtime instances

This is the part that shows real understanding of C++ object lifetime and architecture.

## ASCII flow

### Global module flow

```text
[ex00] -> [ex01] -> [ex02] -> [ex03]
   |         |         |         |
   v         v         v         v
 polymorphism  deep copy  abstract class  interfaces + cloning
```

### ex03 object flow

```text
+------------------+
|  MateriaSource   |
|  learns materias |
+------------------+
          |
          | createMateria(type)
          v
+------------------+
|     clone()      |
|  new AMateria*   |
+------------------+
          |
          v
+------------------+
|    Character     |
|  equip / use     |
+------------------+
          |
          v
+------------------+
|     Target       |
| receives effect  |
+------------------+
```

### Ownership summary

```text
Dog / Cat
  |-- own Brain (ex01, ex02)
  |-- destroy Brain in destructor

MateriaSource
  |-- own learned AMateria templates
  |-- clone when creating a new materia

Character
  |-- own equipped AMateria pointers
  |-- release them in destructor
```

## Implementation notes

- Keep the code compatible with C++98.
- Prefer explicit Rule of Three implementations where heap memory exists.
- Always use virtual destructors in polymorphic base classes.
- Avoid shallow copying of owned pointers.
- Keep `makeSound()` and `use()` behavior simple and testable.

## Build per exercise

```text
cd ex00 && make
cd ex01 && make
cd ex02 && make
cd ex03 && make
```

## Summary

This project is a compact OOP study of how C++ handles object lifetime, inheritance, and polymorphism.

The bonus part is the strongest proof of understanding because it requires correct design, not only correct output.
