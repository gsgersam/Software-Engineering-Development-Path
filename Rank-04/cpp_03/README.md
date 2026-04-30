# CPP_03 - Inheritance in C++

A comprehensive C++ project demonstrating object-oriented programming concepts through single and multiple inheritance patterns. This project is part of École 42's Common Core curriculum.

## Table of Contents

- [Overview](#overview)
- [Project Structure](#project-structure)
- [Exercises](#exercises)
- [Bonus](#bonus)
- [Compilation & Usage](#compilation--usage)
- [Key Concepts](#key-concepts)

---

## Overview

This project progressively introduces inheritance mechanisms in C++ by building a robot combat system. Each exercise adds complexity and new features to reinforce essential OOP principles.

**Target Language:** C++98

---

## Project Structure

```
cpp_03/
├── ex00/                 # Exercise 0 - Basic Class
├── ex01/                 # Exercise 1 - Single Inheritance
├── ex02/                 # Exercise 2 - Single Inheritance (Different Implementation)
├── ex03/                 # Exercise 3 - Multiple Inheritance (BONUS)
├── README.md
└── DEVDOC.md
```

---

## Exercises

### Exercise 0: ClapTrap - The Base Class

**Objective:** Master constructors, destructors, and basic class member functions.

**Key Features:**
- Default and parameterized constructors
- Copy constructor and assignment operator
- Destructors with proper resource management
- Member functions: `attack()`, `takeDamage()`, `beRepaired()`

**Class Attributes:**
- `_name` (std::string)
- `_HitPoints` (10)
- `_EnergyPoints` (10)
- `_AttackDamage` (0)

**Rules of the Five:**
- Constructor
- Copy Constructor
- Assignment Operator
- Destructor
- Destructor Chaining

---

### Exercise 1: ScavTrap - Single Inheritance

**Objective:** Implement single inheritance and method overriding.

**Key Features:**
- Inherits from `ClapTrap`
- Introduces `public` inheritance
- Overrides `attack()` method with custom behavior
- Adds exclusive ability: `guardGate()`

**Class Attributes (Inherited + Modified):**
- `_HitPoints`: 100
- `_EnergyPoints`: 50
- `_AttackDamage`: 20

**New Ability:**
- `guardGate()`: Puts the guard in "Gate keeper" mode

---

### Exercise 2: FragTrap - Another Single Inheritance Branch

**Objective:** Explore multiple implementations of single inheritance with different characteristics.

**Key Features:**
- Inherits from `ClapTrap`
- Independent implementation from `ScavTrap`
- Overrides `attack()` method
- Adds exclusive ability: `highFivesGuys()`

**Class Attributes (Inherited + Modified):**
- `_HitPoints`: 100
- `_EnergyPoints`: 100
- `_AttackDamage`: 30

**New Ability:**
- `highFivesGuys()`: Allows the robot to request a high-five

---

## Bonus: Exercise 3 - DiamondTrap - Multiple Inheritance

**Objective:** Master multiple inheritance patterns and handle the diamond problem using virtual inheritance.

**Key Features:**
- Multiple inheritance from both `ScavTrap` and `FragTrap`
- Combines abilities from both parent classes
- Solves the diamond problem through proper virtual inheritance
- Demonstrates method resolution order (MRO)

**Class Attributes:**
- `_HitPoints`: 100 (from FragTrap)
- `_EnergyPoints`: 50 (from ScavTrap)
- `_AttackDamage`: 30 (from FragTrap)
- Exclusive `_name` attribute to avoid naming conflicts

**New Ability:**
- `whoAmI()`: Displays the robot's identity and lineage

**Inherited Abilities:**
- `attack()` from ScavTrap
- `guardGate()` from ScavTrap
- `highFivesGuys()` from FragTrap
- `takeDamage()` and `beRepaired()` from ClapTrap

---

## Compilation & Usage

### Building Individual Exercises

```bash
cd ex00
make
./ClapTrap

cd ../ex01
make
./ScavTrap

cd ../ex02
make
./FragTrap

cd ../ex03
make
./DiamondTrap
```

### Cleaning Build Files

```bash
make clean       # Remove object files
make fclean      # Remove object files and executable
make re          # Rebuild from scratch
```

### Example Output

```
===== [TEST 1] Default Constructor =====
ClapTrap Default constructor called
...
===== [TEST 2] Named Constructor =====
ClapTrap parameterized constructor called
ClapTrap "Warrior" created with 10 HP, 10 EP, 0 AD
```

---

## Key Concepts

### 1. **Single Inheritance**
   - Child class inherits public members and methods from parent
   - Method overriding allows specialized behavior
   - Constructor chaining ensures proper initialization

### 2. **Multiple Inheritance**
   - A class can inherit from multiple base classes
   - Enables combining features from different sources
   - Requires careful handling of the diamond problem

### 3. **The Diamond Problem**
   - Occurs when a class inherits from two classes with a common ancestor
   - Solved using **virtual inheritance** (C++98 style)
   - Ensures single instance of base class data

### 4. **Rules of the Five (C++98)**
   - `Constructor`
   - `Copy Constructor`
   - `Assignment Operator`
   - `Destructor`
   - Proper resource management and cleanup

### 5. **Polymorphism & Method Overriding**
   - Virtual functions allow different behavior based on actual object type
   - Enables flexible code that works with base class pointers/references
   - Base class pointers can point to derived objects

### 6. **Member Accessibility**
   - `private`: Only accessible within the class
   - `public`: Accessible from anywhere
   - `protected`: Accessible within the class and derived classes

---

## Compilation Flags

All exercises compile with strict C++ standards:

```
-Wall       # All warnings
-Werror     # Treat warnings as errors
-Wextra     # Extra warnings
-std=c++98  # C++98 standard
```

---

## Author

**Developed by:** gsanin-m  
**School:** École 42

---

## Learning Outcomes

Upon completion, you will understand:
- ✓ Object-oriented programming principles
- ✓ Inheritance hierarchies and polymorphism
- ✓ Constructor and destructor chaining
- ✓ Virtual functions and method overriding
- ✓ Multiple inheritance and the diamond problem
- ✓ The Rules of the Five in C++98
- ✓ Memory management and resource cleanup
- ✓ Proper class design patterns

---

## Notes

- All code follows École 42's coding standards (42 header style)
- Destructors are marked virtual for proper polymorphic cleanup
- Copy semantics are properly implemented to avoid shallow copying issues
- This project uses C++98 features exclusively
