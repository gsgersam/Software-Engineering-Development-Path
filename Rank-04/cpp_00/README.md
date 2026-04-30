# C++ Module 00 — Workspace Overview

This repository contains the 42 `cpp_00` exercises:

- `ex00`: `megaphone`
- `ex01`: `phonebook`
- `ex02`: `account`

All exercises are built with:

- `c++`
- `-Wall -Wextra -Werror`
- `-std=c++98`

---

## Project Structure

```text
cpp_00/
├── ex00/
│   ├── makefile
│   └── megaphone.cpp
├── ex01/
│   ├── makefile
│   ├── main.cpp
│   ├── contact.hpp
│   ├── contact.cpp
│   ├── phonebook.hpp
│   └── phonebook.cpp
└── ex02/
    ├── makefile
    ├── Account.hpp
    ├── account.cpp
    └── tests.cpp
```

---

## Build & Run

### ex00

```bash
cd ex00
make
./megaphone "hello" "42"
```

### ex01

```bash
cd ex01
make
./phonebook
```

### ex02

```bash
cd ex02
make
./account
```

---

## Mandatory vs Bonus

### ex00 — Megaphone

**Mandatory**
- Print `* LOUD AND UNBEARABLE FEEDBACK NOISE *` if no argument is provided.
- Otherwise, print all arguments converted to uppercase.

**Bonus / Extra behavior in this implementation**
- Safe uppercase conversion using `unsigned char` cast before `toupper`.

### ex01 — PhoneBook

**Mandatory**
- Interactive commands: `ADD`, `SEARCH`, `EXIT`.
- Store up to 8 contacts.
- Overwrite oldest contact when full.
- Show fixed-width table for search and display full contact by index.

**Bonus / Extra behavior in this implementation**
- Empty-field validation for all inputs.
- Digit-only validation for phone numbers.
- Colored terminal output (ANSI sequences).
- Graceful handling of EOF (`Ctrl+D`) during input.

### ex02 — Account

**Mandatory**
- Implement account lifecycle and logging API required by `tests.cpp`.
- Maintain global account statistics (accounts, total amount, deposits, withdrawals).
- Print timestamped logs for account creation, deposit, withdrawal, status, and closure.

**Bonus / Extra behavior in this implementation**
- No separate official bonus target/file is provided in this workspace.
- The current code focuses on mandatory behavior and expected output format.

---

## ASCII Flow (High-Level)

### ex00 flow

```text
+------------------+
| Start program    |
+--------+---------+
         |
         v
+--------------------------+
| argc == 1 ?              |
+-----+--------------------+
      |Yes                 |No
      v                    v
+----------------------+   +-------------------------------+
| Print default noise  |   | For each argv[i] (i >= 1):    |
+----------+-----------+   |   For each char: uppercase    |
           |               |   Print char                  |
           v               +---------------+---------------+
+----------------------+                   |
| End                  |<------------------+
+----------------------+
```

### ex01 flow

```text
+-----------------------+
| Start phonebook       |
+-----------+-----------+
            |
            v
+-------------------------------+
| Read command (ADD/SEARCH/EXIT)|
+------+------------------------+
       |
       +--> ADD ----> Collect validated fields --> Save contact --> Loop
       |
       +--> SEARCH -> If empty: message -----------> Loop
       |                    |
       |                    v
       |              Print table -> Read valid index -> Show full info -> Loop
       |
       +--> EXIT ---> Print exit message -> End
       |
       +--> Other --> Print invalid command -> Loop
```

### ex02 flow

```text
+------------------------------+
| tests.cpp creates accounts   |
+--------------+---------------+
               |
               v
+------------------------------+
| Constructor updates globals  |
| and prints creation log      |
+--------------+---------------+
               |
               v
+------------------------------+
| displayAccountsInfos/status  |
+--------------+---------------+
               |
               v
+------------------------------+
| Deposit loop                 |
| - update account amount      |
| - update global counters     |
| - print deposit log          |
+--------------+---------------+
               |
               v
+------------------------------+
| Withdrawal loop              |
| - if insufficient: refused   |
| - else update values/counters|
| - print withdrawal log       |
+--------------+---------------+
               |
               v
+------------------------------+
| End of main -> destructors   |
| print closure logs           |
+------------------------------+
```
