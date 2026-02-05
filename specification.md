# Aspire programming language specification

This is a specification for a language that aspires to be my ideal language.

It seeks to follow the ideal espoused by the Scheme reports. To quote R3RS:

> Programming languages should be designed not by piling feature on top of feature, but by removing the weaknesses and restrictions that make additional features appear necessary.

## Primary features

- Closures: Run-time abstractions
  - Kernel-style operatives and applicatives
- Macros: Compile-time abstractions
  - Closures that take and return syntax objects that have been designated for use as macros.
- Reader extensions: Read-time abstraction
  - Closures that take strings and return objects that have been designated for use as reader extensions.
- Delimited continuations: Composable framework for control mechanisms

Note that closures are not exclusively evaluated at run-time. An implementation is allowed and encouraged to evaluate closures at compile-time when the data it depends on is available at compile time. Compile-time data will be available to run-time code having been initialized to the values they had at the end of the compile phase.

## Secondary features

Features which may need to be built-in for the implementation and use of primary features.

- First class environments
- Symbols
- A primitive evaluation applicative

## Tertiary features

These are features that could be implemented using the primary features but which will be built-in for practical reasons.

- Persistent List-like Data (PLD): These will be the basic data structure in the same way that lists are the basic data structure in Lisp. (Most likely RRB trees)
- Numbers and arithmetic using a Scheme-like numerical tower
- Strings and Booleans
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

## Quaternary features

If these can be efficiently implemented as libraries, they do not need to be built-in features.

- Gradual static typing
- Generic functions
- Modules

## Phases

Aspire programs potentially move through three phases. e.g.

- Read time
- Compile time (which may consist of multiple levels)
- Run time

Things are not entirely straight-forward, however. Read-time can require reading, compiling, and running reader extensions. Compile-time can require reading, compiling, and running macros. Run-time can do JIT compilation.

## Closure phase annotations

Implementations are free to and encouraged to evaluate closures during the earliest phase possible. A closure can optionally be annotated with which phases it is intended to be used in. e.g. A programmer may write a closure that they intend to only be evaluated at compile-time. They can annotate the code to express that so that the implementation can flag when the programmer uses it in a way that cannot be used at compile-time.
