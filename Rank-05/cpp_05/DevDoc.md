# DevDoc - ex03 Intern Form Factory

Technical design reference for `ex03`.

## Contents

- [Goal](#goal)
- [Architecture](#architecture)
- [Control flow](#control-flow)
- [Memory ownership](#memory-ownership)
- [Exception model](#exception-model)
- [Build settings](#build-settings)
- [How to extend](#how-to-extend)

## Goal

`ex03` introduces a factory-style `Intern` into the Module 05 architecture (`Bureaucrat` + `AForm` hierarchy).

Core objective:

- create concrete `AForm` instances from string input without exposing form construction to callers.

## Architecture

### `AForm` (abstract base)

Responsibilities:

- stores `_name`, `_isSigned`, `_gradeToSignIt`, `_gradeRequiredToExecuteIt`,
- validates grade bounds in constructor,
- signs through `beSigned(Bureaucrat&)`,
- protects execution through `execute(const Bureaucrat&)`,
- delegates concrete behavior via pure virtual `executeAction()`.

Execution guard order:

1. reject if form is not signed,
2. reject if executor grade is too low,
3. run `executeAction()`.

### Concrete forms

| Class | Sign grade | Execute grade | `executeAction()` behavior |
|---|---:|---:|---|
| `ShrubberyCreationForm` | 145 | 137 | Creates `<target>_shrubbery` and writes ASCII tree |
| `RobotomyRequestForm` | 72 | 45 | Uses `rand() % 2` for success/failure after drill sound |
| `PresidentialPardonForm` | 25 | 5 | Prints pardon message |

### `Bureaucrat`

Responsibilities:

- stores `_name` and `_grade` (valid range: `1..150`),
- signs forms via `signForm(AForm&)`,
- executes forms via `executeForm(const AForm&)`,
- catches and prints operation exceptions.

### `Intern`

Factory API:

- `AForm* makeForm(std::string formName, std::string target)`

Implementation pattern:

- resolver maps form name to id,
- `switch` allocates concrete object with `new`,
- unknown name throws `FormNotFoundException`.

Accepted input names:

- `shrubbery creation`
- `robotomy request`
- `presidential pardon`

## Control flow

`main.cpp` runtime sequence:

1. show menu loop,
2. read `formName` and `target`,
3. create `Bureaucrat("Boss", 1)`,
4. create `Intern`,
5. call `Intern::makeForm(...)`,
6. call `signForm` then `executeForm`,
7. delete returned pointer.

Reference flow diagram: [FLOW_ASCII.md](FLOW_ASCII.md)

## Memory ownership

- `Intern::makeForm` returns heap memory (`new`).
- Caller owns returned `AForm*` and must `delete` it.
- Current `main.cpp` correctly releases `formPtr` after use.

## Exception model

Primary custom exceptions in this exercise:

- `AForm::GradeTooHighException`
- `AForm::GradeTooLowException`
- `AForm::FormNotSignItException`
- `AForm::GradeTooLowToExecute`
- `AForm::FileIOException`
- `Bureaucrat::GradeTooHighException`
- `Bureaucrat::GradeTooLowException`
- `Intern::FormNotFoundException`

## Build settings

- compiler: `c++`
- flags: `-Wall -Werror -Wextra -std=c++98`
- binary: `Bureaucrat`

## How to extend

To add a new form type:

1. create a new class inheriting from `AForm`,
2. implement constructor thresholds + `executeAction()`,
3. include the header where needed,
4. add mapping in `Intern` resolver + `switch`,
5. update [README.md](README.md), this DevDoc, and flow docs.
