# Aspire programming language specification

TODO: Examples throughout

QUESTION: Could this be summarized as "Kernel plus macros, reader extensions, and RRB trees"?

THOUGHT: Should impure function be required to be marked as impure?

THOUGHT: Have a general sequence type that gets optimized into a string behind the scenes when possible? The way a number can seamlessly go from fixnum to bignum and back, a sequence can go from string to sequence and back without the programmer having to (in most cases) treat them as different types.

This is a specification for a language that aspires to be my ideal language.

It seeks to follow the ideal espoused by the Scheme reports. To quote R3RS:

> Programming languages should be designed not by piling feature on top of feature, but by removing the weaknesses and restrictions that make additional features appear necessary.

Aspire is not meant, however, to be a purely theoretical and perfectly elegant language. For that would be a weakness itself. Aspire aims to be performant and minimize the costs of its abstractions.

There is one weakness Aspire accepts, however. The trade-off of a highly extensible language is a proliferation of dialects.

## Requirements

Aspire must be...

- Memory-safe
- Suitable for general-purpose use
- Suitable for systems programming

Aspire does not have built-in concurrency, so concurrency safety depends upon libraries that implement it.

## Primary features

These are the "big ideas" of the language. The small set of powerful features that can be used to build other features.

- Closures: Run-time abstractions
  - Kernel-style operatives and applicatives created from operatives with a primitive `wrap` operator.
- Macros: Compile-time abstractions
  - Closures that take and return syntax objects that have been designated for use as macros.
- Reader extensions: Read-time abstraction
  - Closures that take strings and return objects that have been designated for use as reader extensions.
- Delimited continuations: Composable framework for control mechanisms
- Foreign Function Interface: Provides a single mechanism to access host system functionality that would otherwise need to be built-in

A single abstraction method, closures, is provided along with mechanisms to use it at read-time, compile-time, and run-time.

The notion of operative and applicatives from Kernel allows special forms to be written in the language. Unlike macros, operatives can selectively evaluate operands in the caller's environment or compute environments to evaluate in.

Operatives, however, can be a poor substitute for macros as macros are easier to optimize.

Conceptually, applicatives are simply operatives that evaluate their arguments. Implementation experience has convinced this author, however, that Kernel's approach of a primitive `wrap` operation to convert an operative into an applicative is a better approach that trying to make the `wrap` operation a library feature.

Note that closures are not exclusively evaluated at run-time. An implementation is allowed and encouraged to evaluate closures at compile-time when the data it depends on is available at compile time. Compile-time data will be available to run-time code having been initialized to the values they had at the end of the compile phase. This removes the weakness of the programmer having to manually optimize such situations.

Continuations provide the ability to implement powerful control mechanisms such as exceptions, coroutines, and coöperative multitasking as libraries. Delimited continuations address issues with undelimited continuations and provide composability so that different continuation-based mechanisms play well together.

Any list of the disadvantages of Lisp lists the S-expression syntax. S-expressions are a practical choice for the native syntax of the language, but Aspire should enable alternative syntaxes to be as close to native as possible. Thus, reader extensions are provided as a primary feature.

## Secondary features

Features which required for the primary features but which have applications beyond that.

- A primitive evaluation applicative
  - Required for the operative/applicative approach
- First class environments
  - Required for the operative/applicative approach
- Symbols
  - Required for environments which map symbols to values
- Syntax objects
  - Provides a value (such as a symbol) with lexical information and source-location for a value
  - Necessary to enable hygienic syntax manipulations
- Persistent List-like Data (PLD)
  - The native data structure akin to lists in Lisp.
  - More friendly to modern computers than singly-linked lists.
  - Required to represent compound objects such as syntax objects.
  - Most likely implemented as RRB trees.
- Strings
  - Required for reader extensions

## Tertiary features

These are features that could conceptually be implemented using the primary features but which would be too inefficient or otherwise impractical.

- Numbers and arithmetic using a Scheme-like numerical tower
  - Church numerals or other workarounds perform poorly and fail to take advantage of the numeric support built nearly every processor available.
- Equality
  - Follow the Clojure model...
    - `=` for structural equality
    - `identical?` for optimized identity check
    - Possibly `==` for cross-type numeric comparisons
  - Equality testing is a ubiquitous, critical, and performance sensitive feature.
    - The `identical?` predicate can be implemented as a simple pointer comparison.
    - The `=` predicate benefits from type-specific optimizations.
    - Structural equality benefits from implementation-level optimizations.
- Booleans
  - There's no real benefit to deferring all Booleans to user code.
  - The big downside of no built-in Boolean type is inconsistency and confusion about such a ubiquitous concept. The majority of control structures should agree on true/false/truthy/falsy for the sake of programmer sanity.
  - Built-in Booleans do not prevent user-specified true/false/truthy/falsy concepts from being used where appropriate in special circumstances.
- Mutable boxes
  - Values in Aspire are immutable.
  - Generally, name/value bindings in environments are also immutable.
  - A name in an environment, however, can be bound to mutable box. The box holds a value, and the value of the mutable box can be changed.
  - While there are benefits to a language with no mutablility, it forces programs into awkward workarounds. Limited mutability allows more straight-forward implementation of some computations without opening the door to the downsides of unlimited mutability.
- Hash maps
  - Maps are a ubiquitous programming concept, and the O(1) lookup of hash maps is essential to practical programming.
  - Environments are typically implemented using hash maps internally. Since the implementation likely already contains a hash map implementation, it makes sense to expose it for general use.

Possible tertiary features to consider:

- Nil/null
- Characters

## Potential library features

Features that should be buildable as Aspire libraries (with the use of FFI when necessary). This specification will not cover implementations of these features. If anything here can not be efficiently implemented as a library under this specification, the specification should be improved to remove that weakness.

- I/O primitives (via FFI)
- Concurrency primitives (via FFI)
- Error/exceptions (via continuations)
- Pattern matching (via macros)
- Generic functions (via operatives/macros)
- Modules
  - First-class environments, macros, and compile-time I/O should make modules implementable as a library.
  - There may be a need for some minimal module-like functionality for bootstrapping.
  - Implementation experience will determine whether this should be promoted to a tertiary feature.
- Gradual static typing

## Syntax

The native syntax will be S-expression based. Although the point or reader extensions is to enable support for other syntaxes to make them as close to native as possible.

Aspire expressions will use the Lisp-style of either *atom* or `(`*operation-expression* *operand-expressions* *...*`)`. When the latter is evaluated, the *operator-expression* is evaluated. If it evaluates to an operative, then the *operand-expressions* are passed to the operative unevaluated along with the dynamic environment. If it is an applicative, the *operand-expressions* are evaluated before being passed.

## Phases

Aspire programs potentially move through three phases. e.g.

- Read time
- Compile time (which may consist of multiple levels)
- Run time

Things are not entirely straight-forward, however. Read-time can require reading, compiling, and running reader extensions. Compile-time can require reading, compiling, and running macros. Run-time can do JIT compilation.

While I/O at read or compile-time is allowable, the programmer should have to be explicit when I/O is expected to happen in the compilation environment.

## Closure phase annotations

Implementations are free to and encouraged to evaluate closures during the earliest phase possible. A closure can optionally be annotated with which phases it is intended to be used in. e.g. A programmer may write a closure that they intend to only be evaluated at compile-time. They can annotate the code to express that so that the implementation can flag when the programmer uses it in a way that cannot be used at compile-time.
