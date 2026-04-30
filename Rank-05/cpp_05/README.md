# C++ Module 05 - ex03: Intern

This exercise extends the Bureaucrat/Form system by introducing an `Intern` class that creates concrete forms dynamically from a string.

## Contents

- [Overview](#overview)
- [Build](#build)
- [Run](#run)
- [Supported forms](#supported-forms)
- [Error handling](#error-handling)
- [Flow](#flow)
- [Developer documentation](#developer-documentation)

## Overview

The program:

- shows an interactive menu,
- asks the `Intern` to create a form,
- uses `Bureaucrat("Boss", 1)` to sign and execute it,
- reports failures through exceptions.

> Form names must match exactly one of the supported strings.

## Build

From `ex03/`:

```bash
make
```

Binary:

```bash
./Bureaucrat
```

Cleanup:

```bash
make clean
make fclean
make re
```

## Run

```bash
./Bureaucrat
```

Interactive steps:

1. Choose `1` to execute a form.
2. Enter the form name.
3. Enter the target.

## Supported forms

| Input name | Concrete type | Sign grade | Execute grade | Effect |
|---|---|---:|---:|---|
| `shrubbery creation` | `ShrubberyCreationForm` | 145 | 137 | Creates `<target>_shrubbery` and writes an ASCII tree |
| `robotomy request` | `RobotomyRequestForm` | 72 | 45 | Prints drilling noise, then random success/failure |
| `presidential pardon` | `PresidentialPardonForm` | 25 | 5 | Prints pardon message |

## Error handling

Main failure cases handled by the current implementation:

- unknown form name (`Intern::FormNotFoundException`),
- invalid bureaucrat/form grades,
- execution of unsigned form,
- executor grade too low,
- file I/O error while writing shrubbery output.

## Flow

Complete ASCII flow: [FLOW_ASCII.md](FLOW_ASCII.md)

Plain text version: [FLOW_ASCII.txt](FLOW_ASCII.txt)

Quick flow:

```text
START
  |
  v
Menu [1 run / 2 quit]
  |                \
  |2                \1
  v                  v
 EXIT        Read formName + target
                    |
                    v
        Intern.makeForm(formName, target)
          | known              | unknown
          v                    v
      AForm* created      exception -> catch -> menu
          |
          v
   Bureaucrat.signForm(*form)
          |
          v
 Bureaucrat.executeForm(*form)
          |
          v
       delete form
          |
          v
       menu (loop)
```

## Developer documentation

Detailed design notes: [DevDoc.md](DevDoc.md)
