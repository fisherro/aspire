# Concerns

This document servers as a to-do list and topics of concerns to consider and a place to consider them.

## Macros

Macros allow run time abstractions to be run at compile time.

Aside: Both operatives and applicatives are closures, right? I should change the spec to reflect that.

Ideally, the programmer doesn't have to think about when the expansion happens. It's up to the implementation to optimize things. An expression like `(define x (5 * 1024))` can be done at compile time without the programmer having to specify that it should. Likewise, a sequence like `(define y 5)(define x (y * 1024))` could also be evaluated at compile time. The same should apply to macros.

Perhaps the programmer has a way to annotate an expression to say that he wants it to happen at a specific phase, but this should be a case of the programmer asking the implementation to enforce his intention and tell the programmer if they made a mistake that prevents that. It shouldn't be the programmer telling the implementation how to optimize things because optimization is the job of the implementation.

The only restriction is that compile-time code can't rely on run-time data.

I/O at compile-time: We shouldn't forbid it because I can see how it would be useful. I/O code run at compile time would need to know if it is being run in the compile-time or run-time environment, though. Perhaps this is a case where a programmer would want to use annotations to indicate which environment is expected. Maybe compile-time I/O is only allowed in code annotated as compile-time only.

Note that the same code could be evaluated at compile-time, it could be just-in-time compiled at run-time, or it could be evaluated at run-time.

There may be a need for explicit syntax objects to enable hygiene.

Do reader extensions need to know what phase they're running in? Can there be a "mini-reader" phase inside another phase?

## Phases

- Read time
- Compile time
- Link time
- Module load time
- JIT time
- Run time

Each phase could be multistage.

Could possibly be collapsed to...

- Read time
- Multiple compile time depths
- Run time

## Syntax

## Garbage collection and/or memory safety

## Concerns & Questions

**Name**: Both candidates have issues. "Ideal" conflicts with existing software. "Aspire" works but may sound presumptuous. Consider names that reflect the core philosophy (composability, abstraction layers, etc.).

**Fexprs are controversial**: While powerful, they have well-documented problems:

- Break referential transparency
- Make optimization extremely difficult
- Hygiene is hard to achieve
- The relationship with your macro system needs clarification—if macros are built on fexprs, how do you get hygiene?

**RRB trees as *the* basic structure**: This seems overly specific. Lists work in Lisp because they're simple and match the syntax. What's your rationale? RRB trees are efficient but complex. Does this align with your minimalism goal?

**Missing critical details**:

- Syntax/surface language design (S-expressions? Something else?)
- Evaluation semantics (eager? lazy? something else?)
- Module/namespace system
- Scoping rules (especially important with fexprs)
- Memory model for concurrency

**Secondary features bloat**: Your secondary list is quite long. Some items (pattern matching, exceptions) might be tertiary if they can be built with delimited continuations. Hash maps might be derivable from RRB trees.

**Gradual typing as tertiary**: This is extremely complex to get right. Either commit to it as a core feature or explicitly defer it. "If it can be efficiently implemented" is optimistic for type systems.

**Pattern matching placement**: Given your primary features, pattern matching could potentially be a compile-time abstraction (macro), suggesting it belongs in tertiary.

## Recommendations

1. Add a section on evaluation strategy and semantics
2. Specify module system design
3. Justify the fexpr choice and address known criticisms
4. Clarify the relationship between fexprs, macros, and hygiene
5. Consider whether RRB trees should be *a* data structure rather than *the* basic one
6. Reduce secondary features by moving more to tertiary
7. Add syntax examples to make the specification more concrete
8. Consider implementation feasibility—which features genuinely compose vs. which need deep integration?
