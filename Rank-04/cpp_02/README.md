# CPP Module 02: Fixed-Point Numbers

## Overview

This project implements a **Fixed-Point Number** class in C++ with progressive complexity across four exercises. Fixed-point arithmetic is used when you need decimal precision without floating-point overhead, commonly used in embedded systems, game engines, and financial calculations.

## Project Structure

```
cpp_02/
├── ex00/  - Basic Fixed class with raw bit management
├── ex01/  - Type conversions (int/float to Fixed and vice versa)
├── ex02/  - Operator overloading (arithmetic & comparison)
└── ex03/  - Bonus: Triangle point containment (BSP algorithm)
```

---

## Exercise Breakdown

### **Ex00: Basic Fixed-Point Class**

Implements core Fixed-point functionality:
- Default constructor, copy constructor, and assignment operator
- Destructor
- `getRawBits()` - returns the raw fixed-point value
- `setRawBits()` - sets the raw fixed-point value
- Static fractional bits constant (typically 8)

**Key Concepts:**
- Fixed-point representation: `value = raw_bits / 2^fractional_bits`
- Shallow vs. deep copying in C++

---

### **Ex01: Type Conversions**

Extends Fixed class with conversion capabilities:
- Constructor from `int`
- Constructor from `float`
- `toFloat()` - converts Fixed to float
- `toInt()` - converts Fixed to int
- Overloaded insertion operator `<<` for stream output

**Key Concepts:**
- Type conversion constructors
- Operator overloading for I/O
- Precision handling in conversions

---

### **Ex02: Operator Overloading**

Implements complete operator support:

**Comparison Operators:**
- `>`, `<`, `>=`, `<=`, `==`, `!=`

**Arithmetic Operators:**
- `+`, `-`, `*`, `/`

**Increment/Decrement Operators:**
- Pre-increment: `++fixed`
- Post-increment: `fixed++`
- Pre-decrement: `--fixed`
- Post-decrement: `fixed--`

**Static Methods:**
- `min(const Fixed& a, const Fixed& b)` - returns reference to minimum
- `max(const Fixed& a, const Fixed& b)` - returns reference to maximum

**Key Concepts:**
- Operator overloading syntax and conventions
- Difference between pre/post increment operators
- Static methods for utility functions

---

### **Ex03: Bonus - Point in Triangle (BSP Algorithm)**

**Files:**
- `Point.hpp` / `Point.cpp` - 2D point representation using Fixed coordinates
- `bsp.cpp` - Binary Space Partitioning algorithm

**Functionality:**
```cpp
bool bsp(Point const a, Point const b, Point const c, Point const point);
```

Determines if a point lies **strictly inside** a triangle (not on edges).

**Algorithm:**
1. Calculate signed area between point and each edge
2. Check if all areas have the **same sign** (inside)
3. Verify point is **not on any edge** (signed areas ≠ 0)

**Key Concepts:**
- Geometric algorithms
- Cross-product for area calculation
- Edge case handling (points on edges)

---

## Compilation & Testing

Each exercise has its own makefile:

```bash
# Build
cd ex00 && make

# Run
./fixed

# Clean
make clean
make fclean
```

---

## Fixed-Point Arithmetic Explanation

### Representation
- **Raw bits**: Integer storage (typically `int`)
- **Fractional bits**: Number of decimal bits (typically 8)

### Formula
```
Fixed Value = Raw Bits / 2^Fractional_Bits
```

### Example (8 fractional bits)
```
Raw Bits = 256  →  Fixed = 256 / 256 = 1.0
Raw Bits = 512  →  Fixed = 512 / 256 = 2.0
Raw Bits = 768  →  Fixed = 768 / 256 = 3.0
Raw Bits = 384  →  Fixed = 384 / 256 = 1.5
```

### Advantages
- ✓ Precise decimal arithmetic
- ✓ No floating-point rounding errors
- ✓ Faster than float in some systems
- ✓ Deterministic behavior

### Disadvantages
- ✗ Limited range
- ✗ Requires careful scaling during operations
- ✗ Division can lose precision

---

## Technical Details

### Fractional Bits Constant
All exercises use: `static const int b = 8;`

This means:
- 8 bits reserved for fractional part
- Remaining bits for integer part
- Default value on construction: 0

### Copy Constructor & Assignment
- Deep copy of raw bits
- Necessary for proper object management

### Stream Output Format
Fixed-point numbers are displayed as their floating-point equivalent:
```cpp
Fixed num(5);
std::cout << num;  // Output: 5
```

---

## Algorithm Flow: Fixed-Point Conversion

```
┌─────────────────────────────────────────┐
│   Input: float value f                  │
└──────────────────┬──────────────────────┘
                   │
                   ▼
        ┌──────────────────────┐
        │ Calculate:           │
        │ raw = f * 2^b        │
        │ (b = fractional bits)│
        └──────────────┬───────┘
                       │
                       ▼
        ┌──────────────────────┐
        │ Store in Fixed class │
        │ as integer raw_bits  │
        └──────────────┬───────┘
                       │
                       ▼
        ┌──────────────────────┐
        │ Scale down when      │
        │ converting back to   │
        │ float: f = raw/2^b  │
        └──────────────────────┘
```

---

## Bonus: Triangle Point Containment (Ex03)

### Algorithm Flow

```
┌─────────────────────────────────────────────┐
│ Input: Triangle (A, B, C) & Point P         │
└──────────────────┬──────────────────────────┘
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
  Calculate          Calculate
  SignedArea(A,B,P)  SignedArea(B,C,P)
        │ area1              │ area2
        │                    │
        └──────────┬─────────┘
                   │
                   ▼
          Calculate SignedArea(C,A,P)
                area3
                   │
        ┌──────────┴──────────┐
        │                     │
        ▼                     ▼
  Check Same Sign   Check Not On Edge
  All positive OR     All areas ≠ 0
  All negative        (area1 != 0 &&
                       area2 != 0 &&
                       area3 != 0)
        │                     │
        └──────────┬──────────┘
                   │
                   ▼
         ┌──────────────────┐
         │ P Inside Triangle?│
         │ same_sign &&      │
         │ not_on_edge       │
         └────────┬─────────┘
                  │
        ┌─────────┴─────────┐
        │                   │
        ▼                   ▼
      true                false
    (Inside)          (Outside/OnEdge)
```

### Signed Area Formula

```
SignedArea(P1, P2, P3) = (P1.x - P3.x) * (P2.y - P3.y) 
                         - (P2.x - P3.x) * (P1.y - P3.y)
```

This uses the **cross product** principle to detect which side of a line a point lies on.

---

## Key Learning Outcomes

✓ Object-oriented programming in C++  
✓ Operator overloading conventions  
✓ Copy construction and assignment  
✓ Fixed-point arithmetic principles  
✓ Geometric algorithms (bonus)  
✓ Memory management and lifecycle  

---

## Author

**gsanin-m** @ 42 School  
Created: September 2025

---

## Notes

- All classes follow 42 School's C++ standards
- Memory must be properly managed (no leaks)
- Code must follow Orthodox Canonical Form (OCF)
- Makefiles must support `make`, `make clean`, and `make fclean`
