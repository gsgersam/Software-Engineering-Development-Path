# ex03 Execution Flow (ASCII)

```text
EX03 EXECUTION FLOW (ASCII)
===========================

+----------------------+
|        START         |
+----------+-----------+
           |
           v
+------------------------------+
| Show menu: [1] Run [2] Quit  |
+---------------+--------------+
                |
        +-------+--------+
        |                |
        v                v
   [input=2]        [input=1]
        |                |
        v                v
+---------------+   +-------------------------+
|      EXIT     |   | Read formName + target |
+---------------+   +-----------+-------------+
                            |
                            v
                +---------------------------+
                | Create Bureaucrat("Boss",1) |
                +-------------+-------------+
                              |
                              v
                     +----------------+
                     | Create Intern  |
                     +--------+-------+
                              |
                              v
          +-------------------------------------------+
          | formPtr = Intern.makeForm(formName,target)|
          +-------------------+-----------------------+
                              |
                +-------------+--------------------------+
                |                                        |
                v                                        v
      [known form name]                         [unknown name]
                |                                        |
                v                                        v
 +----------------------------------+        +---------------------------+
 | new concrete form (AForm*)       |        | throw FormNotFoundException|
 | - ShrubberyCreationForm          |        +-------------+-------------+
 | - RobotomyRequestForm            |                      |
 | - PresidentialPardonForm         |                      v
 +----------------+-----------------+           +------------------------+
                  |                             | catch std::exception   |
                  v                             | print error            |
      +-----------------------------+           +-----------+------------+
      | Bureaucrat.signForm(*form)  |                       |
      +--------------+--------------+                       |
                     |                                      |
                     v                                      |
      +-----------------------------+                       |
      | Bureaucrat.executeForm(*form)|                      |
      +--------------+--------------+                       |
                     |                                      |
                     v                                      |
      +-----------------------------+                       |
      | delete formPtr; formPtr=0  |<----------------------+
      +--------------+--------------+
                     |
                     v
             +---------------+
             | Back to menu  |
             +-------+-------+
                     |
                     v
                  (loop)
```

## Concrete `executeAction` details

- `ShrubberyCreationForm`: creates `<target>_shrubbery` and writes an ASCII tree.
- `RobotomyRequestForm`: prints drilling sound and then random success/failure (`rand() % 2`).
- `PresidentialPardonForm`: prints `"<target> has been pardoned by Zaphod Beeblebrox"`.
