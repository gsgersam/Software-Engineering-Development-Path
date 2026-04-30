# Developer Documentation (DEVDOC)

This document explains implementation details, data flow, and design decisions for `cpp_00`.

---

## 1) Technical Context

- Language: C++98
- Compiler flags: `-Wall -Wextra -Werror -std=c++98`
- Build system: per-exercise `makefile`
- Architecture style: small isolated console programs per exercise

---

## 2) ex00 — `megaphone`

### Source
- `ex00/megaphone.cpp`

### Core logic
- If no user arguments exist (`argc == 1`), print a default noise sentence.
- Otherwise iterate arguments from index 1 to `argc - 1`.
- Convert each character to uppercase and stream to stdout.

### Design notes
- Uses `std::string` to process each argument.
- Uses `toupper(static_cast<unsigned char>(c))` for safer character conversion.
- Complexity: O(total input characters).

### Mandatory vs Bonus
**Mandatory**
- Default message when no arguments.
- Uppercase output for provided arguments.

**Bonus / Extra**
- Defensive cast pattern before `toupper`.

### ASCII flow
```text
[Start]
   |
   v
[argc == 1?] --yes--> [Print default noise] --> [End]
   |
   no
   v
[Loop args]
   |
   v
[Loop chars -> uppercase -> print]
   |
   v
[Print newline]
   |
   v
[End]
```

---

## 3) ex01 — `phonebook`

### Sources
- `ex01/main.cpp`
- `ex01/phonebook.hpp`
- `ex01/phonebook.cpp`
- `ex01/contact.hpp`
- `ex01/contact.cpp`

### Class model

#### `Contact`
Private fields:
- `firstName`
- `lastName`
- `nickname`
- `phoneNumber`
- `darkestSecret`

Private helper:
- `truncate(const std::string&) const`: trims text to width 10 (`9 chars + '.'`).

Public methods:
- `handleContacts()` : collect validated fields from input.
- `displayContactInfo(int index) const` : compact table row.
- `displayFullInfo() const` : full record print.

#### `PhoneBook`
Private fields:
- `Contact contacts[8]`
- `int count`
- `int oldest_index`

Public methods:
- `addContact()` : writes in ring-buffer style.
- `getValidIndex() const` : robust integer index input.
- `searchContacts() const` : list contacts and show selected full details.

### Input validation behavior
- Empty fields are rejected and re-requested.
- Phone number accepts only digit characters.
- `Ctrl+D` (EOF) is detected in input routines and exits gracefully.

### Ring buffer overwrite policy
- Once 8 contacts exist, next `ADD` overwrites at `oldest_index`.
- `oldest_index = (oldest_index + 1) % 8` after every add.

### Mandatory vs Bonus
**Mandatory**
- Commands `ADD`, `SEARCH`, `EXIT`.
- Store up to 8 contacts and overwrite oldest when full.
- Indexed table display and full contact display.

**Bonus / Extra**
- ANSI-colored terminal output.
- Additional validation and user-friendly error prompts.
- Strong EOF handling in both command and field input paths.

### ASCII flow
```text
[Start]
   |
   v
[Print menu + read command]
   |
   +--> ADD ----> [Collect validated fields] ---> [Store contact] ---> (loop)
   |
   +--> SEARCH -> [count == 0?]
   |                 |yes
   |                 v
   |             [No contacts] -------------------------------> (loop)
   |                 |
   |                 no
   |                 v
   |            [Print table] -> [Read valid index] -> [Print full info] -> (loop)
   |
   +--> EXIT ---> [Exit message] ---> [End]
   |
   +--> other --> [Invalid command] --------------------------> (loop)
```

---

## 4) ex02 — `account`

### Sources
- `ex02/Account.hpp`
- `ex02/account.cpp`
- `ex02/tests.cpp`

### Static/global state managed by `Account`
- `_nbAccounts`
- `_totalAmount`
- `_totalNbDeposits`
- `_totalNbWithdrawals`

### Per-account state
- `_accountIndex`
- `_amount`
- `_nbDeposits`
- `_nbWithdrawals`

### Implemented API behavior
- Constructor logs creation with timestamp and updates global state.
- Destructor logs closure and adjusts global total amount.
- `makeDeposit` updates both local and global counters.
- `makeWithdrawal` refuses when insufficient funds; otherwise updates all counters.
- `displayStatus` and `displayAccountsInfos` print current state snapshots.

### Timestamp format
- `_displayTimestamp()` prints in the form: `[YYYYMMDD_HHMMSS]`.

### Mandatory vs Bonus
**Mandatory**
- Full API contract matching `tests.cpp` usage.
- Consistent updates of static totals and per-account counters.
- Timestamped logs for all required actions.

**Bonus / Extra**
- No explicit bonus target exists in the provided `ex02` workspace files.

### ASCII flow
```text
[Start tests main]
   |
   v
[Create vector<Account>]
   |
   v
[displayAccountsInfos + displayStatus(all)]
   |
   v
[For each account: makeDeposit]
   |
   v
[displayAccountsInfos + displayStatus(all)]
   |
   v
[For each account: makeWithdrawal]
   |
   v
[displayAccountsInfos + displayStatus(all)]
   |
   v
[Program ends -> destructors run]
   |
   v
[End]
```

---

## 5) Known Limits / Notes

- `ex01` command parsing is case-sensitive (`ADD`, `SEARCH`, `EXIT` only).
- `ex01` uses ANSI escape colors; output appearance depends on terminal support.
- `ex02` output correctness is tied to expected logging format in the provided test.
