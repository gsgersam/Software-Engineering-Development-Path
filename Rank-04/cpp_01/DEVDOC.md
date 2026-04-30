# CPP Module 01 - Developer Documentation

## 1) Technical Goal
This module practices foundational C++98 concepts:
- Object lifetime and deterministic destruction
- Stack vs heap allocation
- References vs pointers
- Basic composition and ownership assumptions
- Function dispatch strategies

## 2) Project Map
- `ex00`: Zombie class + factory/free helper
- `ex01`: Zombie array (horde) creation API
- `ex02`: Pointer/reference comparison demo
- `ex03`: Weapon ownership semantics through two human classes
- `ex04`: Manual string replacement engine over full file content
- `ex05`: Message dispatch via array of member-function pointers
- `ex06`: Filtering behavior via index lookup + switch fallthrough

## 3) Mandatory vs Bonus Delimitation

### Mandatory (present)
All current code under `ex00` through `ex06` is treated as mandatory implementation.

### Bonus (present state)
No separate bonus implementation exists in this repository (no bonus folders, bonus targets, or `_bonus` sources).

### Recommended packaging for evaluation
- Submit `ex00` to `ex06` as mandatory.
- Mark bonus as "not provided" unless your school rubric defines another split.

## 4) Internal Design Notes

### ex00 - Zombie single instances
- `newZombie(name)` allocates on heap and returns pointer.
- `randomChump(name)` creates stack object and announces immediately.
- Destructor prints destruction message, making lifetime visible in output.

### ex01 - Zombie horde
- `zombieHorde(N, name)` allocates `N` zombies with `new[]`.
- Names are assigned with `setName`, currently randomized from a static pool.
- Caller owns memory and must use `delete[]`.
- `N <= 0` returns `NULL`.

### ex02 - Pointer and reference demo
- Single `std::string` object.
- One pointer and one reference bound to the same object.
- Prints addresses and values to show identity and access equivalence.

### ex03 - Weapon and humans
- `Weapon` stores mutable `type` and exposes `getType` + `setType`.
- `HumanA` requires a `Weapon&` at construction (always armed).
- `HumanB` stores `Weapon*` and can be unarmed until `setWeapon`.
- Demonstrates reference mandatory binding vs pointer optional binding.

### ex04 - Replace engine
- Reads entire file into memory via `ostringstream`.
- `ftReplace` scans left-to-right; when `s1` matches at index, appends `s2`.
- Empty `s1` returns original text unchanged (guard against infinite replacement).
- Writes output to `<input>.replace`.

### ex05 - Harl dispatcher
- Maps level strings to member-function pointers:
  - `DEBUG`, `INFO`, `WARNING`, `ERROR`
- Executes only exact matching level handler.
- Unknown level silently does nothing.

### ex06 - Harl filter
- Finds selected level index in ordered array.
- Uses switch fallthrough to print selected level and all higher severities.
- Unknown level prints custom fallback message.

## 5) ASCII Flow (Detailed: ex06 HarlFilter)

```text
+--------------------------------------+
| main(argc, argv)                     |
+--------------------------------------+
                |
                v
+--------------------------------------+
| argc == 2 ?                          |
+-------------------+------------------+
        Yes         |        No
                    v
         +-----------------------------+
         | Print usage + return 1      |
         +-----------------------------+

Yes branch:
                |
                v
+--------------------------------------+
| level = argv[1]                      |
| create Harl instance                 |
+--------------------------------------+
                |
                v
+--------------------------------------+
| Harl::complain(level)                |
+--------------------------------------+
                |
                v
+--------------------------------------+
| Find level index i in                |
| [DEBUG, INFO, WARNING, ERROR]        |
+-------------------+------------------+
        Found       |     Not found
                    v
         +-----------------------------+
         | Print fallback message      |
         +-----------------------------+

Found branch:
                |
                v
+--------------------------------------+
| switch(i) with fallthrough           |
| i=0 -> DEBUG, INFO, WARNING, ERROR   |
| i=1 -> INFO, WARNING, ERROR          |
| i=2 -> WARNING, ERROR                |
| i=3 -> ERROR                         |
+--------------------------------------+
                |
                v
+--------------------------------------+
| return 0                             |
+--------------------------------------+
```

## 6) Build Contract
Each exercise has a local `makefile` with standard targets:
- `all`
- `clean`
- `fclean`
- `re`

Compiler profile is consistently C++98 with strict warnings as errors.

## 7) Maintenance Tips
- Keep stack/heap ownership explicit in function names and call sites.
- Avoid adding hidden global state; pass dependencies directly.
- Preserve exercise isolation (`ex00`..`ex06`) to simplify correction.
- If adding bonus later, place it in explicit `bonus` folders or `_bonus` files to make evaluation unambiguous.
