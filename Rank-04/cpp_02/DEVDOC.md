# CPP Module 02: Development Documentation

## Table of Contents

1. [Architecture Overview](#architecture-overview)
2. [Detailed Implementation](#detailed-implementation)
3. [Class Specifications](#class-specifications)
4. [Algorithm Implementation](#algorithm-implementation)
5. [Development Flow](#development-flow)
6. [Testing Strategy](#testing-strategy)
7. [Edge Cases & Considerations](#edge-cases--considerations)

---

## Architecture Overview

### Module Composition

```
┌────────────────────────────────────────────────────┐
│                  CPP Module 02                     │
├────────────────────────────────────────────────────┤
│                                                    │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐      │
│  │   Ex00   │   │   Ex01   │   │   Ex02   │      │
│  │          │   │          │   │          │      │
│  │  Fixed   │──▶│  Fixed + │──▶│  Fixed + │      │
│  │ (Basic)  │   │Convert   │   │Operators │      │
│  └──────────┘   └──────────┘   └──────────┘      │
│                                       │           │
│                                       ▼           │
│                                  ┌──────────┐    │
│                                  │   Ex03   │    │
│                                  │          │    │
│                                  │ Fixed +  │    │
│                                  │  Point + │    │
│                                  │   BSP    │    │
│                                  └──────────┘    │
│                                  (BONUS)         │
│                                                    │
└────────────────────────────────────────────────────┘
```

### Design Pattern: Orthodox Canonical Form (OCF)

All classes implement:
```
✓ Default constructor
✓ Copy constructor
✓ Assignment operator
✓ Destructor
✓ Optionally: other constructors (conversion)
```

---

## Detailed Implementation

### Ex00: Basic Fixed-Point Class

#### File Structure
```
ex00/
├── Fixed.hpp       (Class declaration)
├── Fixed.cpp       (Implementation)
├── main.cpp        (Test/demonstration)
└── makefile
```

#### Implementation Details

**Fixed.hpp**
```cpp
class Fixed {
private:
    int a;                          // Raw bits storage
    static const int b = 8;         // Fractional bits (8)

public:
    Fixed(void);                    // Default: a = 0
    Fixed(const Fixed& a);          // Copy constructor
    Fixed& operator=(const Fixed& a);
    ~Fixed(void);
    
    int getRawBits(void) const;
    void setRawBits(int const raw);
};
```

**Raw Bit Storage Calculation:**
```
When Fixed x = 5:
  x.getRawBits() returns 1280 (5 * 256)
  where 256 = 2^8

When Fixed y = 2.5:
  y.getRawBits() returns 640 (2.5 * 256)
```

#### Development Flow for Ex00

```
START: Create Fixed class
       │
       ▼
   Add private members (a, b)
       │
       ▼
   Implement default constructor
   (Initialize a = 0)
       │
       ▼
   Implement destructor
   (Empty, no dynamic allocation)
       │
       ▼
   Implement copy constructor
   (Deep copy of a)
       │
       ▼
   Implement assignment operator
   (Copy a, return *this)
       │
       ▼
   Implement getRawBits() & setRawBits()
       │
       ▼
   Test with main.cpp
       │
       ▼
   END: Verify output matches subject
```

---

### Ex01: Type Conversion

#### New Components

**Convert from int:**
```cpp
Fixed(const int n) {
    a = n * (1 << b);  // n * 2^8
}
```

**Convert from float:**
```cpp
Fixed(const float f) {
    a = roundf(f * (1 << b));  // round(f * 2^8)
}
```

**Convert to int:**
```cpp
int toInt(void) const {
    return a / (1 << b);  // a / 2^8 (truncates)
}
```

**Convert to float:**
```cpp
float toFloat(void) const {
    return (float)a / (1 << b);  // a / 2^8
}
```

#### Stream Operator
```cpp
std::ostream& operator<<(std::ostream& out, const Fixed& fixed) {
    out << fixed.toFloat();
    return out;
}
```

#### Conversion Flow

```
┌──────────────────────────────────────┐
│  User Creates: Fixed x(3.14f)        │
└─────────────────┬────────────────────┘
                  │
                  ▼
      ┌───────────────────────────┐
      │ Conversion Constructor:   │
      │ Fixed(const float f)      │
      └───────────────┬───────────┘
                      │
                      ▼
          ┌───────────────────────┐
          │ Calculate:            │
          │ a = roundf(3.14 * 256)│
          │ a = roundf(803.84)    │
          │ a = 804               │
          └───────────────┬───────┘
                          │
                          ▼
          ┌───────────────────────┐
          │ Stored in Fixed class │
          │ raw_bits = 804        │
          └───────────────┬───────┘
                          │
                ┌─────────┴──────────┐
                │                    │
        toFloat()                 toInt()
                │                    │
                ▼                    ▼
          804 / 256            804 / 256
          = 3.140625           = 3 (integer)
```

#### Development Flow for Ex01

```
START: Extend Ex00
       │
       ▼
   Add int conversion constructor
   Test: Fixed(42) → getRawBits() = 10752
       │
       ▼
   Add float conversion constructor
   Test: Fixed(3.14f) → getRawBits() = 804
       │
       ▼
   Add toFloat() method
   Test: toFloat() → 3.14...
       │
       ▼
   Add toInt() method
   Test: toInt() → 3
       │
       ▼
   Implement << operator
   Test: std::cout << Fixed(2.5) → "2.5"
       │
       ▼
   END: All floating-point operations work
```

---

### Ex02: Comprehensive Operator Overloading

#### Comparison Operators

**Implementation Pattern:**
```cpp
bool Fixed::operator>(const Fixed& other) const {
    return a > other.getRawBits();
}
```

All comparison operators work directly on raw bits since they represent the same scale.

| Operator | Implementation |
|----------|---|
| `>` | `a > other.a` |
| `<` | `a < other.a` |
| `>=` | `a >= other.a` |
| `<=` | `a <= other.a` |
| `==` | `a == other.a` |
| `!=` | `a != other.a` |

#### Arithmetic Operators

**Addition & Subtraction:**
```cpp
Fixed Fixed::operator+(const Fixed& other) const {
    Fixed result;
    result.setRawBits(a + other.getRawBits());
    return result;
}
```

**Multiplication:**
```cpp
Fixed Fixed::operator*(const Fixed& other) const {
    Fixed result;
    long long temp = ((long long)a * other.getRawBits()) / (1 << b);
    result.setRawBits((int)temp);
    return result;
}
```
⚠️ Uses `long long` to prevent overflow during multiplication.

**Division:**
```cpp
Fixed Fixed::operator/(const Fixed& other) const {
    Fixed result;
    long long temp = ((long long)a * (1 << b)) / other.getRawBits();
    result.setRawBits((int)temp);
    return result;
}
```

#### Increment/Decrement Operators

**Post-increment (returns old value):**
```cpp
Fixed Fixed::operator++(int) {
    Fixed tmp(*this);      // Copy current
    a++;                   // Increment raw bits by 1
    return tmp;            // Return old value
}
```

**Pre-increment (returns new value):**
```cpp
Fixed& Fixed::operator++() {
    a++;
    return *this;
}
```

Similar pattern for decrement operators.

#### Static Min/Max Methods

```cpp
const Fixed& Fixed::min(const Fixed& a, const Fixed& b) {
    return (a < b) ? a : b;
}

const Fixed& Fixed::max(const Fixed& a, const Fixed& b) {
    return (a > b) ? a : b;
}
```

Returns **reference** to avoid unnecessary copying.

#### Operator Overloading Flow

```
┌─────────────────────────────────┐
│  Operations on Fixed Numbers    │
└────────────────┬────────────────┘
                 │
    ┌────────────┼────────────┐
    │            │            │
    ▼            ▼            ▼
 Compare      Arithmetic   Increment
    │            │            │
    ├─ >       ├─ + (add raw) ├─ ++ (pre)
    ├─ <       ├─ - (sub raw) ├─ ++ (post)
    ├─ >=      ├─ * (scale)   ├─ -- (pre)
    ├─ <=      └─ / (scale)   └─ -- (post)
    ├─ ==
    └─ !=
```

#### Development Flow for Ex02

```
START: Extend Ex01
       │
       ▼
   Implement comparison operators (6 total)
   Test: a > b, a <= b, etc.
       │
       ▼
   Implement addition operator
   Test: Fixed(2) + Fixed(3) = Fixed(5)
       │
       ▼
   Implement subtraction operator
   Test: Fixed(5) - Fixed(2) = Fixed(3)
       │
       ▼
   Implement multiplication operator
   ⚠️ Use long long to prevent overflow
   Test: Fixed(2) * Fixed(3) = Fixed(6)
       │
       ▼
   Implement division operator
   ⚠️ Use long long to prevent overflow
   Test: Fixed(6) / Fixed(2) = Fixed(3)
       │
       ▼
   Implement post-increment
   Create temporary copy, return old value
   Test: a = x++
       │
       ▼
   Implement pre-increment
   Increment directly, return reference
   Test: a = ++x
       │
       ▼
   Implement post/pre decrement
   Same pattern as increment
       │
       ▼
   Implement static min() & max()
   Return const reference
       │
       ▼
   END: All operators fully functional
```

---

### Ex03: Bonus - Point in Triangle (BSP)

#### Point Class

**File Structure**
```
ex03/
├── Fixed.hpp, Fixed.cpp    (From Ex02)
├── Point.hpp, Point.cpp    (New - 2D point)
├── bsp.cpp                 (New - Algorithm)
├── main.cpp
└── makefile
```

**Point.hpp**
```cpp
class Point {
private:
    Fixed const x;      // Immutable coordinates
    Fixed const y;

public:
    Point();
    Point(const float x, const float y);
    Point(const Point& other);
    Point& operator=(const Point& other);
    ~Point();
    
    Fixed getX() const;
    Fixed getY() const;
};
```

**Key Design Decision:** x and y are `const` - once set, they cannot change.

#### BSP Algorithm Implementation

**Helper Function:**
```cpp
Fixed signedArea(Point const& p1, Point const& p2, Point const& p3) {
    Fixed term1 = (p1.getX() - p3.getX()) * (p2.getY() - p3.getY());
    Fixed term2 = (p2.getX() - p3.getX()) * (p1.getY() - p3.getY());
    return term1 - term2;
}
```

**Main Algorithm:**
```cpp
bool bsp(Point const a, Point const b, Point const c, Point const point) {
    // Check signed area with each triangle edge
    Fixed area1 = signedArea(a, b, point);  // Edge AB
    Fixed area2 = signedArea(b, c, point);  // Edge BC
    Fixed area3 = signedArea(c, a, point);  // Edge CA
    
    // All positive or all negative = same side
    bool same_sign = (area1 > 0 && area2 > 0 && area3 > 0) 
                  || (area1 < 0 && area2 < 0 && area3 < 0);
    
    // All non-zero = not on edge
    bool not_on_edge = (area1 != 0) && (area2 != 0) && (area3 != 0);
    
    return same_sign && not_on_edge;
}
```

#### BSP Algorithm Deep Dive

**Signed Area Concept:**

The formula calculates a signed area using the cross product:
```
Area = (P1.x - P3.x) * (P2.y - P3.y) - (P2.x - P3.x) * (P1.y - P3.y)
```

This is equivalent to:
```
Area = 2 * Area_of_Triangle(P1, P2, P3)
```

**Sign Meaning:**
- Positive: Point is on the left side of the directed edge
- Negative: Point is on the right side of the directed edge
- Zero: Point lies on the edge

**Algorithm Logic:**
```
If point P is inside triangle ABC:
  - P is on the same side of edge AB (relative to C)
  - P is on the same side of edge BC (relative to A)
  - P is on the same side of edge CA (relative to B)

Same side = signs of all 3 areas match (all positive or all negative)
Not on edge = all areas are non-zero
```

#### BSP Detection Flow

```
INPUT: Triangle vertices (A, B, C) & Test Point P
       │
       ▼
┌──────────────────────────────────────┐
│ Calculate signed area between:       │
│ - Edge AB and point P                │
│ - Edge BC and point P                │
│ - Edge CA and point P                │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│ All 3 areas positive? → same side ✓  │
│            OR                        │
│ All 3 areas negative? → same side ✓  │
└──────────────────────────────────────┘
       │
       ▼
┌──────────────────────────────────────┐
│ Any area = 0? → Point on edge ✗      │
│                                      │
│ All non-zero? → Not on edge ✓        │
└──────────────────────────────────────┘
       │
       ▼
  same_sign && not_on_edge
       │
    ┌──┴──┐
    │     │
   YES   NO
    │     │
    ▼     ▼
  INSIDE OUTSIDE
```

#### Development Flow for Ex03

```
START: Extend Ex02
       │
       ▼
   Create Point class with immutable x, y
   Implement constructors & getters
       │
       ▼
   Implement signedArea() helper function
   Test area calculation with known triangle
       │
       ▼
   Implement bsp() main function
   Test with point inside triangle → true
       │
       ▼
   Test with point outside triangle → false
       │
       ▼
   Test edge case: point on edge → false
       │
       ▼
   Test edge case: point at vertex → false
       │
       ▼
   END: BSP algorithm fully working
```

---

## Class Specifications

### Fixed Class Hierarchy

```
┌─────────────────────────────────────┐
│ Fixed (All Exercises)               │
├─────────────────────────────────────┤
│ PRIVATE:                            │
│  - int a                            │
│  - static const int b = 8           │
├─────────────────────────────────────┤
│ PUBLIC:                             │
│                                     │
│ EX00:                               │
│  + Fixed()                          │
│  + Fixed(const Fixed&)              │
│  + Fixed& operator=(const Fixed&)   │
│  + ~Fixed()                         │
│  + int getRawBits() const           │
│  + void setRawBits(int)             │
│                                     │
│ + EX01:                             │
│  + Fixed(const int)                 │
│  + Fixed(const float)               │
│  + int toInt() const                │
│  + float toFloat() const            │
│  + friend << operator               │
│                                     │
│ + EX02:                             │
│  + bool operator>  / <  / >= / <=   │
│  + bool operator== / !=             │
│  + Fixed operator+ / - / * / /      │
│  + Fixed operator++(int) [post]     │
│  + Fixed& operator++() [pre]        │
│  + Fixed operator--(int) [post]     │
│  + Fixed& operator--() [pre]        │
│  + static const Fixed& min(...)     │
│  + static const Fixed& max(...)     │
│                                     │
└─────────────────────────────────────┘
```

### Point Class (Ex03 Bonus)

```
┌─────────────────────────────────────┐
│ Point                               │
├─────────────────────────────────────┤
│ PRIVATE:                            │
│  - Fixed const x;                   │
│  - Fixed const y;                   │
├─────────────────────────────────────┤
│ PUBLIC:                             │
│  + Point()                          │
│  + Point(const float, const float)  │
│  + Point(const Point&)              │
│  + Point& operator=(const Point&)   │
│  + ~Point()                         │
│  + Fixed getX() const               │
│  + Fixed getY() const               │
│                                     │
└─────────────────────────────────────┘
```

---

## Algorithm Implementation

### Signed Area Calculation

**Mathematical Basis (Shoelace Formula):**

For triangle vertices P1, P2, P3:
```
Area = |1/2 * det([P1.x - P3.x, P1.y - P3.y],
                   [P2.x - P3.x, P2.y - P3.y])|

     = |1/2 * ((P1.x - P3.x) * (P2.y - P3.y) - (P1.y - P3.y) * (P2.x - P3.x))|
```

We use the **signed** version (without absolute value) to detect point location.

### Why Fixed-Point Arithmetic Matters in BSP

```
✓ Exact: No rounding errors in area calculation
✓ Consistent: Same result every execution
✓ Deterministic: No floating-point quirks
✗ Precision: Limited by bit width (typically 32-bit int)
```

---

## Testing Strategy

### Ex00 Test Cases

```cpp
Fixed x;              // Default: 0
Fixed y(x);           // Copy: 0
y = x;                // Assignment works
std::cout << x.getRawBits();  // Should be 0
x.setRawBits(256);
std::cout << x.getRawBits();  // Should be 256
```

### Ex01 Test Cases

```cpp
Fixed a(5);           // int conversion: raw = 1280
Fixed b(2.5f);        // float conversion: raw = 640
std::cout << a;       // "5"
std::cout << b;       // "2.5"
std::cout << a.toInt();    // 5
std::cout << b.toFloat();  // 2.5
```

### Ex02 Test Cases

```cpp
Fixed a(2.5f), b(1.5f);

// Comparison
a > b;              // true
a <= b;             // false
a == b;             // false

// Arithmetic
a + b;              // Fixed(4)
a - b;              // Fixed(1)
a * b;              // Fixed(3.75)
a / b;              // Fixed(1.667...)

// Increment/Decrement
a++;                // After: a.toFloat() = 2.50390625 (1/256 increment)
++b;                // Before: b.toFloat() = 1.50390625
--a;                // a back to 2.5

// Min/Max
Fixed::min(a, b);   // b (1.5)
Fixed::max(a, b);   // a (2.5)
```

### Ex03 Test Cases (Bonus)

```cpp
Point a(0.0f, 0.0f);
Point b(4.0f, 0.0f);
Point c(0.0f, 4.0f);

Point inside(1.5f, 1.5f);
Point outside(5.0f, 5.0f);
Point onEdge(2.0f, 0.0f);

bool test1 = bsp(a, b, c, inside);   // true
bool test2 = bsp(a, b, c, outside);  // false
bool test3 = bsp(a, b, c, onEdge);   // false
```

---

## Edge Cases & Considerations

### 1. **Overflow in Multiplication/Division**

**Problem:**
```cpp
Fixed big(100);
Fixed result = big * big;  // 100 * 100 = 10000
// Raw: 25600 * 25600 = 655,360,000
// Exceeds int range!
```

**Solution:**
```cpp
Fixed Fixed::operator*(const Fixed& other) const {
    long long temp = ((long long)a * other.getRawBits()) / (1 << b);
    return Fixed(temp / 256.0f);  // Convert back safely
}
```

### 2. **Division by Zero**

**Problem:**
```cpp
Fixed zero(0);
Fixed result = something / zero;  // Undefined behavior!
```

**Solution:** Add guard clause (if implementing error handling):
```cpp
Fixed Fixed::operator/(const Fixed& other) const {
    if (other.getRawBits() == 0)
        throw std::runtime_error("Division by zero");
    // ...
}
```

### 3. **Point on Triangle Edge**

**Problem:**
```cpp
Point p(2.0f, 0.0f);  // Lies on edge from (0,0) to (4,0)
bool inside = bsp(a, b, c, p);
// signedArea = 0 for one edge
// not_on_edge = false
// Result: false ✓ (correctly excluded)
```

### 4. **Point at Triangle Vertex**

**Problem:**
```cpp
Point p = a;  // Same as vertex A
bool inside = bsp(a, b, c, p);
// At least one signedArea = 0
// Result: false ✓ (correctly excluded)
```

### 5. **Floating-Point to Fixed Precision Loss**

**Problem:**
```cpp
Fixed x(1.0f / 3.0f);  // 0.333333...
// Float: 0.333333 (limited precision)
// Fixed(8 bits): raw = 85 (closest integer)
// Actual value: 85/256 = 0.3320...
// Lost precision from 0.333333 → 0.3320
```

**Mitigation:**
```cpp
Fixed(const float f) {
    a = roundf(f * (1 << b));  // Use rounding for best fit
}
```

### 6. **Comparison with Zero**

```cpp
Fixed x(0.0f);
x == 0;  // Compare raw bits: 0 == 0 ✓

Fixed y(0.000001f);  // Extremely small but not zero
y == 0;  // Potentially problematic!
// Raw: roundf(0.000001 * 256) = 0
// Result: y == 0 (true)
// But y should be slightly above zero!
```

---

## Development Checklist

### Ex00
- [ ] Default constructor initializes `a = 0`
- [ ] Copy constructor performs deep copy
- [ ] Assignment operator returns `*this`
- [ ] Destructor is empty/trivial
- [ ] getRawBits() returns correct raw value
- [ ] setRawBits() updates raw value
- [ ] Output matches subject exactly

### Ex01
- [ ] int constructor: `a = n << b` (multiply by 256)
- [ ] float constructor: `a = roundf(f * (1 << b))`
- [ ] toInt() returns `a >> b` (divide by 256)
- [ ] toFloat() returns `(float)a / (1 << b)`
- [ ] << operator outputs as float
- [ ] All Ex00 methods still work

### Ex02
- [ ] All 6 comparison operators work correctly
- [ ] Addition/subtraction add/subtract raw bits
- [ ] Multiplication uses `long long` intermediate
- [ ] Division uses `long long` intermediate
- [ ] Post-increment returns copy with old value
- [ ] Pre-increment returns reference with new value
- [ ] Decrement operators mirror increment
- [ ] min() and max() return const references
- [ ] All Ex01 methods still work

### Ex03 (Bonus)
- [ ] Point class has immutable x and y
- [ ] Point constructors work correctly
- [ ] signedArea() calculates correctly
- [ ] bsp() correctly identifies inside points
- [ ] bsp() correctly identifies outside points
- [ ] bsp() correctly excludes edge points
- [ ] bsp() correctly excludes vertex points

---

## Compilation Notes

### Makefile Variables

```makefile
CC = c++
CFLAGS = -Wall -Wextra -Werror -std=c++98
SRCS = Fixed.cpp main.cpp
OBJS = $(SRCS:.cpp=.o)
TARGET = fixed
```

### Recommended Flags

```
-std=c++98       # Use C++98 standard (42 requirement)
-Wall            # All warnings
-Wextra          # Extra warnings
-Werror          # Treat warnings as errors
-O2              # Optimize (optional)
```

---

## Performance Considerations

### Memory Layout (64-bit system)

```
Fixed instance:
┌──────────────┐
│  int a;      │  4 bytes
│  static b;   │  0 bytes (shared)
└──────────────┘
Total per instance: 4 bytes

Point instance:
┌──────────────┐
│  Fixed x;    │  4 bytes
│  Fixed y;    │  4 bytes
└──────────────┘
Total per instance: 8 bytes
```

### Operation Complexity

```
Basic operations:     O(1) - constant time
- Constructor
- Getter/Setter
- Arithmetic ops
- Comparison ops
- Increment/Decrement

BSP Algorithm:        O(1) - constant time
- 3 area calculations
- Sign comparisons
```

---

## Common Pitfalls

❌ **Mistake 1:** Not using `long long` in multiplication
```cpp
// WRONG
Fixed result;
result.setRawBits(a * other.getRawBits() / (1 << b));  // Overflow!

// CORRECT
long long temp = ((long long)a * other.getRawBits()) / (1 << b);
result.setRawBits((int)temp);
```

❌ **Mistake 2:** Shifting wrong direction in conversions
```cpp
// WRONG
Fixed(const int n) {
    a = n >> b;  // Dividing instead of multiplying!
}

// CORRECT
Fixed(const int n) {
    a = n << b;  // n * 2^8
}
```

❌ **Mistake 3:** Forgetting const on getter methods
```cpp
// WRONG
int getRawBits() {
    return a;
}

// CORRECT
int getRawBits() const {
    return a;
}
```

❌ **Mistake 4:** Not handling post vs pre increment differently
```cpp
// WRONG - Both do the same thing!
Fixed operator++(int) {
    return ++(*this);
}
Fixed& operator++() {
    return ++(*this);
}

// CORRECT - Post returns old value
Fixed operator++(int) {
    Fixed tmp(*this);
    a++;
    return tmp;
}
Fixed& operator++() {
    a++;
    return *this;
}
```

---

## References & Resources

- IEEE 754 Fixed-Point Arithmetic
- Cross Product for Geometry
- Barycentric Coordinates (alternative BSP method)
- C++ Operator Overloading Best Practices

---

## Revision History

| Date | Author | Changes |
|------|--------|---------|
| 2025-09-29 | gsanin-m | Created Ex00-Ex01 |
| 2025-09-30 | gsanin-m | Added Ex02 operators |
| 2025-10-01 | gsanin-m | Added Ex03 BSP bonus |
| 2026-04-22 | DevDoc | Complete development documentation |

---

**Document Version:** 1.0  
**Last Updated:** April 22, 2026  
**Status:** Complete
