# CPP_03 - Developer Documentation

Technical documentation for the Inheritance in C++ project. This guide covers implementation details, architecture, and design patterns used throughout the exercises.

## Table of Contents

- [Architecture Overview](#architecture-overview)
- [Class Hierarchy](#class-hierarchy)
- [Implementation Details](#implementation-details)
- [Method Reference](#method-reference)
- [Best Practices](#best-practices)
- [Troubleshooting](#troubleshooting)
- [Code Examples](#code-examples)

---

## Architecture Overview

### Design Pattern: Template Method & Inheritance Chain

The project follows a hierarchical design where each layer adds specialized behavior:

```
┌────────────────────────────────────────────────────────────┐
│                   ClapTrap (Base Class)                    │
│  ┌──────────────────────────────────────────────────────┐  │
│  │ - _name: std::string                                 │  │
│  │ - _HitPoints: unsigned int                           │  │
│  │ - _EnergyPoints: unsigned int                        │  │
│  │ - _AttackDamage: unsigned int                        │  │
│  │                                                       │  │
│  │ + attack(target)                                     │  │
│  │ + takeDamage(amount)                                 │  │
│  │ + beRepaired(amount)                                 │  │
│  └──────────────────────────────────────────────────────┘  │
└───────────────────┬───────────────────────────────────────┘
                    │
         ┌──────────┴──────────┬─────────────────┐
         │                     │                 │
    ┌────▼──────┐      ┌──────▼─────┐      ┌────▼───────┐
    │ ScavTrap  │      │ FragTrap   │      │ (Virtual)  │
    │ (public)  │      │ (public)   │      │ ClapTrap   │
    ├───────────┤      ├────────────┤      └────────────┘
    │ + attack()│      │ + attack() │
    │ + guard   │      │ + highFive │       DiamondTrap
    │   Gate()  │      │   Guys()   │      (Multiple
    │           │      │            │       Inheritance)
    │ HP: 100   │      │ HP: 100    │
    │ EP: 50    │      │ EP: 100    │      Points From:
    │ AD: 20    │      │ AD: 30     │      • ScavTrap
    └───────────┘      └────────────┘      • FragTrap
         ▲                    ▲
         │                    │
         └────────┬──────────┘
                  │
           ┌──────▼──────┐
           │ DiamondTrap │
           ├─────────────┤
           │ + whoAmI()  │
           │             │
           │ HP: 100     │
           │ EP: 50      │
           │ AD: 30      │
           └─────────────┘
```

---

## Class Hierarchy

### ClapTrap (Base Class)

**Location:** `ex00/`

**Responsibility:** Defines the core robot attributes and basic combat mechanics.

**Key Methods:**

| Method | Purpose | Energy Cost | Notes |
|--------|---------|-------------|-------|
| `ClapTrap()` | Default constructor | - | Initializes with default values |
| `ClapTrap(const std::string& name)` | Parameterized constructor | - | Creates robot with custom name |
| `ClapTrap(const ClapTrap& other)` | Copy constructor | - | Deep copy of another ClapTrap |
| `operator=()` | Assignment operator | - | Assignment of one ClapTrap to another |
| `~ClapTrap()` | Destructor | - | Resource cleanup |
| `attack(target)` | Attack mechanism | 1 EP | Prints attack message, valid only if HP > 0 |
| `takeDamage(amount)` | Receive damage | - | Reduces HP, minimum 0 |
| `beRepaired(amount)` | Repair damage | 1 EP | Increases HP, valid only if HP > 0 |

**Initialization Values:**
```
_HitPoints = 10
_EnergyPoints = 10
_AttackDamage = 0
```

---

### ScavTrap (Exercise 01 - Single Inheritance)

**Location:** `ex01/`

**Inheritance Model:** `public ScavTrap : public ClapTrap`

**Responsibility:** Extends ClapTrap with specialized combat variant focused on defensive capabilities.

**Overridden Methods:**

```cpp
void attack(const std::string& target)
// ScavTrap-specific attack message
// Still costs 1 EP and deals AD damage
```

**New Methods:**

```cpp
void guardGate(void)
// Output: "<name> is now in Gate keeper mode."
// Does NOT cost energy
```

**Modified Attributes:**
```
_HitPoints = 100      (from 10)
_EnergyPoints = 50    (from 10)
_AttackDamage = 20    (from 0)
```

**Constructor Implementation:**
```cpp
ScavTrap::ScavTrap(const std::string& name) : ClapTrap(name)
{
    // Call ClapTrap constructor first
    // Then reset attributes to ScavTrap values
    _HitPoints = 100;
    _EnergyPoints = 50;
    _AttackDamage = 20;
}
```

---

### FragTrap (Exercise 02 - Single Inheritance)

**Location:** `ex02/`

**Inheritance Model:** `public FragTrap : public ClapTrap`

**Responsibility:** Alternative specialized variant with offensive capabilities.

**Overridden Methods:**

```cpp
void attack(const std::string& target)
// FragTrap-specific attack message
// Costs 1 EP, deals AD damage
```

**New Methods:**

```cpp
void highFivesGuys(void)
// Output: "<name> requests everyone for high fives!"
// Does NOT cost energy
```

**Modified Attributes:**
```
_HitPoints = 100      (from 10)
_EnergyPoints = 100   (from 10)
_AttackDamage = 30    (from 0)
```

---

### DiamondTrap (Exercise 03 - Multiple Inheritance BONUS)

**Location:** `ex03/`

**Inheritance Model:** 
```cpp
class DiamondTrap : public ScavTrap, public FragTrap
```

**Responsibility:** Combines strengths from both ScavTrap and FragTrap while resolving ambiguity.

**The Diamond Problem Solution:**

In C++98, virtual inheritance handles this:

```cpp
// Inside ScavTrap header
class ScavTrap : virtual public ClapTrap { ... };

// Inside FragTrap header  
class FragTrap : virtual public ClapTrap { ... };
```

This ensures only ONE instance of ClapTrap data exists in the DiamondTrap object.

**New Methods:**

```cpp
void whoAmI(void)
// Displays: "<DiamondTrap_name> is a DiamondTrap"
// Also displays: "<ClapTrap_name>" (from inherited _name in ClapTrap)
```

**Attribute Resolution:**

```
DiamondTrap::_name           (from DiamondTrap)
ClapTrap::_name              (from ClapTrap, inherited formally)
ClapTrap::_HitPoints = 100   (from FragTrap initialization)
ClapTrap::_EnergyPoints = 50 (from ScavTrap initialization)
ClapTrap::_AttackDamage = 30 (from FragTrap initialization)
```

**Method Resolution Order (MRO):**
```
DiamondTrap::attack()     → ScavTrap::attack()
DiamondTrap::guardGate()  → ScavTrap::guardGate()
DiamondTrap::highFivesGuys() → FragTrap::highFivesGuys()
```

---

## Implementation Details

### Constructor Chaining

**Example: DiamondTrap Constructor**

```cpp
DiamondTrap::DiamondTrap(const std::string& name) 
    : ScavTrap(name), FragTrap(name), _name(name + "_clap_name")
{
    std::cout << "DiamondTrap " << _name << " created!\n";
    // At this point:
    // - ClapTrap::_name was set by ScavTrap constructor
    // - _HitPoints = 100 (from FragTrap)
    // - _EnergyPoints = 50 (overridden from ScavTrap, since it's listed first)
    // - _AttackDamage = 30 (from FragTrap)
    // - DiamondTrap::_name is set to avoid conflict
}
```

### Copy Constructor Implementation

**Pattern Across All Classes:**

```cpp
ClassName::ClassName(const ClassName& other) 
    : ParentClass(other)  // Call parent copy constructor
{
    // Add any class-specific deep copy logic
}
```

### Assignment Operator Implementation

**Pattern Across All Classes:**

```cpp
ClassName& ClassName::operator=(const ClassName& other)
{
    if (this != &other)
    {
        ParentClass::operator=(other);  // Call parent assignment
        // Add any class-specific assignment logic
    }
    return *this;
}
```

### Virtual Destructor Chaining

**Critical for Polymorphism:**

```cpp
// Base class
ClapTrap::~ClapTrap() { ... }  // Virtual destructor

// Derived class
ScavTrap::~ScavTrap() { ... }  // Implicitly virtual

// Multiple inheritance
~DiamondTrap() { ... }  // Ensures proper cleanup order
```

When deleting a `ClapTrap*` pointing to a `ScavTrap`, the destructor chain executes:
1. ScavTrap::~ScavTrap()
2. ClapTrap::~ClapTrap()

---

## Method Reference

### Energy & Combat System

| Action | EP Cost | HP Check | Validity |
|--------|---------|----------|----------|
| `attack()` | 1 | Yes | Only if HP > 0 AND EP > 0 |
| `beRepaired(amount)` | 1 | Yes | Only if HP > 0 AND EP > 0 |
| `takeDamage(amount)` | 0 | - | Always valid |
| `guardGate()` | 0 | - | For ScavTrap, always valid |
| `highFivesGuys()` | 0 | - | For FragTrap, always valid |
| `whoAmI()` | 0 | - | For DiamondTrap, always valid |

### Attack Mechanics

```cpp
void ClapTrap::attack(const std::string& target)
{
    if (_HitPoints == 0)
    {
        std::cout << _name << " is dead, cannot attack!\n";
        return;
    }
    
    if (_EnergyPoints == 0)
    {
        std::cout << _name << " has no energy left!\n";
        return;
    }
    
    _EnergyPoints--;
    std::cout << _name << " attacks " << target 
              << ", causing " << _AttackDamage << " points of damage!\n";
}
```

### Damage System

```cpp
void ClapTrap::takeDamage(unsigned int amount)
{
    if (_HitPoints < amount)
        _HitPoints = 0;      // Prevent negative HP
    else
        _HitPoints -= amount;
    
    std::cout << _name << " takes " << amount 
              << " damage. HP: " << _HitPoints << "\n";
}
```

---

## Best Practices

### 1. Always Call Parent Constructors

```cpp
// CORRECT
ScavTrap::ScavTrap(const std::string& name) : ClapTrap(name)

// WRONG - ClapTrap default constructor called instead
ScavTrap::ScavTrap(const std::string& name)
{
    _name = name;  // ClapTrap not properly initialized
}
```

### 2. Use Virtual Destructors in Base Classes

```cpp
// Base class MUST have virtual destructor
class ClapTrap {
public:
    virtual ~ClapTrap();  // Enables proper polymorphic cleanup
};
```

### 3. Virtual Inheritance for Diamond Problem

```cpp
// ScavTrap.hpp
class ScavTrap : virtual public ClapTrap { ... };

// FragTrap.hpp
class FragTrap : virtual public ClapTrap { ... };

// Ensures single ClapTrap subobject in DiamondTrap
```

### 4. Implement Rules of the Five

```cpp
class ClassName {
public:
    ClassName();                              // Constructor
    ClassName(const ClassName&);              // Copy Constructor
    ClassName& operator=(const ClassName&);   // Assignment Operator
    virtual ~ClassName();                     // Destructor
    
private:
    ClassName& operator=(ClassName&&);        // C++11 (not in this project)
    ClassName(ClassName&&);                   // C++11 (not in this project)
};
```

### 5. Check HP & EP Before Actions

```cpp
// Always validate state before performing actions
if (_HitPoints == 0) return;
if (_EnergyPoints == 0) return;
```

---

## Troubleshooting

### Issue: Ambiguity in Multiple Inheritance

**Problem:** Compiler error when calling inherited methods

```
error: 'ClapTrap' is an ambiguous base of 'DiamondTrap'
```

**Solution:** Use virtual inheritance

```cpp
class ScavTrap : virtual public ClapTrap { };
class FragTrap : virtual public ClapTrap { };
```

### Issue: Destructor Not Called

**Problem:** Memory leaks when using base class pointers

```cpp
ClapTrap* ptr = new ScavTrap("name");
delete ptr;  // Calls only ClapTrap::~ClapTrap()
```

**Solution:** Make destructors virtual

```cpp
class ClapTrap {
public:
    virtual ~ClapTrap();  // Now ScavTrap::~ScavTrap() called too
};
```

### Issue: Wrong Method Called

**Problem:** Child class method overriding not working

**Solution:** Check inheritance keyword

```cpp
// WRONG - Private inheritance doesn't expose parent methods
class ScavTrap : private ClapTrap { };

// CORRECT - Public inheritance
class ScavTrap : public ClapTrap { };
```

---

## Code Examples

### Creating and Using Objects

```cpp
// Exercise 00: Base Class
ClapTrap warrior("Conan");
warrior.attack("enemy");
warrior.takeDamage(5);
warrior.beRepaired(3);

// Exercise 01: ScavTrap
ScavTrap guard("Sentinel");
guard.attack("intruder");
guard.guardGate();

// Exercise 02: FragTrap
FragTrap fragment("Bomber");
fragment.attack("target");
fragment.highFivesGuys();

// Exercise 03: DiamondTrap (BONUS)
DiamondTrap hybrid("Nexus");
hybrid.whoAmI();
hybrid.attack("foe");        // From ScavTrap
hybrid.guardGate();          // From ScavTrap
hybrid.highFivesGuys();      // From FragTrap
hybrid.beRepaired(10);       // From ClapTrap
```

### Polymorphic Behavior

```cpp
// Using base class pointers
ClapTrap* bot1 = new ScavTrap("Guard");
ClapTrap* bot2 = new FragTrap("Attacker");

bot1->attack("enemy");  // Calls ScavTrap::attack()
bot2->attack("enemy");  // Calls FragTrap::attack()

delete bot1;  // Calls ~ScavTrap() → ~ClapTrap()
delete bot2;  // Calls ~FragTrap() → ~ClapTrap()
```

### Testing Copy Semantics

```cpp
ScavTrap original("Alpha");
original.takeDamage(20);

// Copy Constructor
ScavTrap copy = original;  // Creates independent copy

// Assignment Operator
ScavTrap target("Beta");
target = original;         // Replaces target's state
```

---

## Compilation Process

### Step-by-Step Build

```mermaid
Source Files (.cpp)
        ↓
    Preprocessing (includes, macros)
        ↓
    Compilation (→ object files .o)
        ↓
    Linking (combine objects, resolve symbols)
        ↓
    Executable (./ClapTrap, ./ScavTrap, etc.)
```

### Makefile Targets

```makefile
NAME     = Executable name
SRCS     = Source files to compile
OBJS     = Object files generated
CXX      = Compiler (c++)
CXXFLAGS = Compilation flags
```

---

## Testing Checklist

- [ ] Default constructor initializes correct values
- [ ] Parameterized constructor sets custom name
- [ ] Copy constructor creates independent object
- [ ] Assignment operator updates state correctly
- [ ] Destructors print proper messages
- [ ] Attack and damage systems work as expected
- [ ] Energy depletion prevents actions
- [ ] HP = 0 prevents further actions
- [ ] Overridden methods behave correctly
- [ ] Unique abilities work (guardGate, highFivesGuys, whoAmI)
- [ ] Multiple inheritance resolves correctly (DiamondTrap)
- [ ] Polymorphic delete works properly

---

## References

- **C++ Standard:** C++98 (ISO/IEC 14882:1998)
- **Inheritance:** https://en.cppreference.com/w/cpp/language/inheritance
- **Virtual Functions:** https://en.cppreference.com/w/cpp/language/virtual
- **Diamond Problem:** https://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem
- **School:** École 42

