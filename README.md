<img width="96px" src="./dijon.png" align="left">

# Dijon
###### An imperative, stack-oriented esoteric language with highly non-traditional control flow constructs
<br>

## Introduction
Dijon is a language that I initially came up with in 2013.  The original design goal was to create a language with a unique way to handle control flow; in fact, the original vision was to attempt to create a language without loops, if statements, and GOTOs.  The design has gone through many iterations since then, but this is the first completely implemented version.  Between 2019 and 2022, 3 Java versions and 2 Python versions of the interpreter were created, each slightly fleshing out the language and its implementation.  The heart of Dijon is the way it handles memory using both a stack and variables.  However, variables also influence the control flow using a concept called triggers, which are sections of code that execute when the trigger's variable's value is requested.  Variables can be declared in these triggers and are scoped inside them.  Programs may import and use triggers and variables from other files as well, most importantly the standard library.  

## Documentation
Please see the following links to view the language reference, code examples, and future plans.
- [Reference](./REFERENCE.md)
- [Standard Library](./STD_LIBRARY.md)
- [Examples](./examples/)
- [Future features](FUTURE.md)

## Implementation
Please see the reference interpreter, [PyDijon](https://github.com/bluesun212/pydijon), which is written in Python.  
