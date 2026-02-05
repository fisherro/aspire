# Aspire programming language specification

This is a specification for a language that aspires to be my ideal language.

It seeks to follow the ideal espoused by the Scheme reports. To quote R3RS:

> Programming languages should be designed not by piling feature on top of feature, but by removing the weaknesses and restrictions that make additional features appear necessary.

## Primary features

- Closures and Kernel-style fexprs: Run-time abstractions
- Lisp-style macros built on top of fexprs: Compile-time abstractions
- Reader extensions: Read-time abstraction
- Delimited continuations: Composable framework for control mechanisms

## Secondary features

These are features that could be implemented using the primary features but which will be built-in for practical reasons.

- RRB trees will be the basic data structure in the same way that lists are the basic data structure in Lisp.
- Numbers and arithmetic using a Scheme-like numerical tower
- Strings and Booleans
- Symbols
- Hash maps
- Equality
  - Follow the Clojure model...
  - `=` for structural equality
  - `identical?` for optimized identity check
  - Possibly `==` for cross-type numeric comparisons
- I/O primitives
- Exceptions/errors
- Concurrency primitives
- Pattern matching

## Tertiary features

If these can be efficiently implemented as libraries, they do not need to be built-in features.

- Gradual static typing
- Generic functions
