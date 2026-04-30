CPP06 — ASCII Flow
==================

GLOBAL
------

[Start]
   |
   v
+---------------------------------+
| Select exercise: ex00/ex01/ex02 |
+---------------------------------+
   |
   v
+---------------------------------+
| cd ex00 | cd ex01 | cd ex02     |
+---------------------------------+
   |
   v
+---------------------------------+
| make                            |
+---------------------------------+
   |
   v
+---------------------------------+
| Run binary                      |
| ex00: ./convert <literal>       |
| ex01: ./serializer              |
| ex02: ./typeinfo                |
+---------------------------------+
   |
   v
+---------------------------------+
| Observe cast/RTTI behavior      |
+---------------------------------+
   |
   v
[End]


DETAIL BY EXERCISE
------------------

ex00 (ScalarConverter)
literal
  |
  v
classify input
  |
  +--> pseudo-literal -> print pseudo conversions
  +--> scalar literal -> parse -> print char/int/float/double
  +--> invalid        -> print impossible

ex01 (Serializer)
Data* ptr
  |
  v
serialize(ptr) -> uintptr_t raw
  |
  v
deserialize(raw) -> Data* recovered
  |
  v
verify address/value equivalence

ex02 (RTTI)
generate() -> A | B | C
  |
  v
identify(Base*) via dynamic_cast<T*>
  |
  v
identify(Base&) via dynamic_cast<T&> + exceptions
  |
  v
print type + delete instance
