# CPP Module 01

![Language](https://img.shields.io/badge/language-C%2B%2B98-blue)
![Build](https://img.shields.io/badge/build-make-informational)
![Status](https://img.shields.io/badge/status-mandatory%20complete-success)

## Overview
This repository contains C++98 exercises from Module 01, focused on:
- stack vs heap allocation
- pointers and references
- object composition and ownership style
- simple function dispatch patterns

Compiler profile used across exercises:
- `-Wall -Wextra -Werror -std=c++98`

## Table of Contents
- [Project Layout](#project-layout)
- [Mandatory vs Bonus](#mandatory-vs-bonus)
- [Quick Start](#quick-start)
- [Exercise Summary](#exercise-summary)
- [Usage Examples](#usage-examples)
- [ASCII Flow](#ascii-flow)
- [Notes](#notes)

## Project Layout
```text
cpp_01/
├── ex00/
├── ex01/
├── ex02/
├── ex03/
├── ex04/
├── ex05/
└── ex06/
```

## Mandatory vs Bonus

### Mandatory
Implemented exercises:
- `ex00` - Zombie creation on stack and heap (`newZombie`, `randomChump`)
- `ex01` - Zombie horde allocation and cleanup with `new[]` / `delete[]`
- `ex02` - Pointer and reference address/value demonstration
- `ex03` - `Weapon`, `HumanA` (reference), `HumanB` (pointer)
- `ex04` - String replacement in file content (`<filename>.replace` output)
- `ex05` - `Harl` complaint dispatcher using member-function pointers
- `ex06` - `HarlFilter` level-based output using switch fallthrough

### Bonus
No dedicated bonus folder or `*_bonus` target is present in this repository.

Current split for this Git repository:
- Mandatory: fully present in `ex00` to `ex06`
- Bonus: not implemented as separate deliverables

## Quick Start
Run from any exercise directory (`ex00` ... `ex06`):

1. Build: `make`
2. Run: `./<binary_name>`
3. Clean objects: `make clean`
4. Full clean: `make fclean`
5. Rebuild: `make re`

## Exercise Summary

| Exercise | Binary | Focus |
|---|---|---|
| `ex00` | `zombie` | Stack/heap object lifetime |
| `ex01` | `zombieHorde` | Dynamic array allocation |
| `ex02` | `hi` | Pointer vs reference identity |
| `ex03` | `Weapon` | Reference vs pointer composition |
| `ex04` | `replace` | File read/replace/write |
| `ex05` | `Harl` | Member-function pointer dispatch |
| `ex06` | `HarlFilter` | Severity filtering with fallthrough |

## Usage Examples

### ex04
- `./replace text.txt old new`
- Output file: `text.txt.replace`

### ex06
- `./HarlFilter DEBUG`
- `./HarlFilter INFO`
- `./HarlFilter WARNING`
- `./HarlFilter ERROR`

## ASCII Flow

```text
+-----------------------------+
| Start                       |
+-----------------------------+
              |
              v
+-----------------------------+
| Choose exercise (ex00..06)  |
+-----------------------------+
              |
              v
+-----------------------------+
| Enter exercise directory    |
| cd ex0X                     |
+-----------------------------+
              |
              v
+-----------------------------+
| Build with make             |
+-----------------------------+
              |
              v
+-----------------------------+
| Execute binary              |
+-----------------------------+
              |
              v
+-----------------------------------------------+
| Program behavior by exercise                  |
| ex00: single zombie lifecycle                 |
| ex01: horde allocation + announcements        |
| ex02: pointer/reference addresses and values  |
| ex03: HumanA/HumanB attacks with Weapon       |
| ex04: file replace -> .replace output         |
| ex05: exact level complaint                   |
| ex06: filtered level with fallthrough         |
+-----------------------------------------------+
              |
              v
+------------------------------+
| Optional clean (clean/fclean)|
+------------------------------+
              |
              v
+-----------------------------+
| End                         |
+-----------------------------+
```

## Notes
- Some compiled binaries/object files are currently present inside exercise folders.
- For a clean repository history, keep build artifacts untracked and regenerate them with `make`.
