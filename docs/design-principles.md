# Siml Design Principles

Status: Draft  
Last updated: 2026-10-02

## 1. Purpose

Siml means **Simple Language**.

Siml is a systems programming language intended for environments where the compiler itself must be easy to bring up: new operating systems, new instruction sets, experimental processors, soft cores, and other early-stage platforms.

The project is motivated by one of C's most important practical strengths: C can be used very early in the life of a system because a usable compiler can be relatively small.

Siml aims to preserve that strength while giving the language more explicit and predictable semantics than C where doing so does not significantly increase compiler or runtime complexity.

## 2. What "simple" means

"Simple" primarily refers to **compiler implementation complexity**, not merely short syntax.

The central design constraint is:

> A usable Siml compiler must be implementable directly in Assam Core without requiring a complex compiler framework or a complex language runtime.

A minimum compiler should be possible with a straightforward pipeline such as:

```text
source
  -> lexer
  -> parser
  -> name/type checking
  -> small lowering step
  -> Assam Core emission
```

A sophisticated implementation may use better algorithms and optimizations, but those must not become necessary to implement the language correctly.

In particular, the language should not require the bootstrap compiler to have facilities such as:

- a constraint-solving type inference engine;
- mandatory SSA construction;
- mandatory global optimization;
- a garbage collector;
- exception unwinding;
- implicit heap allocation;
- hidden object lifetime machinery;
- dynamic dispatch infrastructure;
- a large mandatory standard runtime.

## 3. Bootstrap is part of the language design

Siml is planned around three compiler implementations.

### Reference compiler

The first compiler is written in a conventional implementation language, initially Go or Rust.

Its purpose is to develop and validate the language quickly.

### Assam Core bootstrap compiler

A minimum Siml compiler is written in Assam Core.

This implementation is not only a port. It is a design test.

If implementing a language feature in Assam Core requires excessive compiler machinery, runtime machinery, or special cases, that is evidence that the feature may be too complex for Siml.

### Self-hosted compiler

A Siml compiler is then written in Siml itself.

This proves that Siml is expressive enough to implement its own toolchain.

The intended bootstrap path is:

```text
Go/Rust simlc
      |
      v
Siml programs
      |
      +----------------------+
      |                      |
      v                      v
Assam Core simlc        Siml simlc
                              |
                              v
                        self-hosting
```

## 4. MVP completion gate

The MVP is considered complete when all of the following are true:

1. A working reference compiler exists in Go or Rust.
2. A usable compiler exists in Assam Core.
3. The Assam Core compiler stays within an explicitly defined complexity budget.
4. A compiler can be written in Siml itself.
5. The Siml compiler can compile the Siml compiler.

The exact numerical complexity budget is still an open design decision.

## 5. Assam Core is the bootstrap complexity boundary

For the MVP, Assam Core is the canonical low-level target and the reference environment for measuring compiler simplicity.

Siml language features should normally lower to small, explicit Assam Core operations or short deterministic sequences.

A feature should be reconsidered if its correct implementation requires the Assam compiler to reconstruct complicated high-level concepts, run expensive global analyses, or rely on a substantial hidden runtime.

This does not require every source construct to map one-to-one to one instruction. It requires the lowering process itself to remain understandable and mechanically implementable.

## 6. Runtime philosophy

Siml must not make the compiler simple by moving equivalent complexity into a mandatory runtime.

The MVP should avoid requiring:

- garbage collection;
- mandatory heap allocation;
- exception unwinding;
- reflection metadata;
- mandatory dynamic dispatch;
- implicit asynchronous runtimes;
- hidden ownership or cleanup machinery.

Small helper routines are acceptable when their contract is explicit and bounded. Platform services such as allocation, I/O, startup, and system calls should have explicit interfaces rather than hidden language behavior.

## 7. Execution performance

Siml does not promise C-equivalent performance.

It should, however, be suitable for operating-system and low-level software on new or experimental architectures. Therefore the language should avoid unnecessary inherent overhead.

A straightforward compiler with little or no optimization should be able to produce reasonably direct code for common operations such as:

- integer arithmetic;
- bit operations;
- branches and loops;
- function calls;
- explicit memory access;
- arrays and records;
- pointer-oriented low-level operations.

Performance that only becomes acceptable after sophisticated optimization is a warning sign for the language design.

Optimizations may be added later, but they are not part of the semantic foundation of the language.

## 8. Compilation performance

Fast compilation is desirable, but it is mainly expected as a consequence of a small compiler and simple language rules.

Siml does not make sophisticated large-project build acceleration a core language requirement.

For example, these may remain responsibilities of a build system or a more advanced compiler implementation:

- parallel compilation;
- dependency caching;
- incremental databases;
- file-system optimization;
- distributed compilation.

If a very large project becomes limited by module count or file I/O, that alone is not a reason to make the core compiler architecture substantially more complex.

## 9. Difference from Fast compile

Siml and Fast compile optimize different things.

**Fast compile** is designed to reduce end-to-end build time, even when doing so requires more sophisticated compiler or build-system machinery. A representative goal is reducing a build that takes one hour with Go to around forty minutes.

**Siml** is designed to reduce the difficulty of implementing and porting the compiler itself.

In short:

```text
Fast compile:
    optimize compilation time

Siml:
    optimize compiler construction simplicity
```

A more complex implementation can be acceptable for Fast compile if it makes builds faster.

For Siml, making the compiler substantially more complex merely to make compilation faster can violate the primary design goal.

## 10. Feature admission questions

Before a feature becomes part of the core language, its design should answer:

1. Can the feature be implemented in the Assam Core bootstrap compiler without exceeding the complexity budget?
2. Can its semantics be explained without relying on sophisticated compiler analysis?
3. Does it lower to explicit low-level behavior?
4. Does it introduce hidden runtime work?
5. Can a minimally optimizing compiler generate reasonably efficient code for it?
6. Can the self-hosted Siml compiler express the implementation clearly?
7. Is the behavior deterministic enough to avoid C-style accidental complexity where a simple rule is possible?

A feature does not need to make every answer trivial, but these questions define the default pressure of the language design.

## 11. Non-goals for the MVP

The MVP is not intended to provide:

- a state-of-the-art optimizing compiler;
- every convenience expected from a modern application language;
- a large managed runtime;
- maximum large-project build performance;
- sophisticated compile-time metaprogramming;
- abstractions whose implementation depends on substantial hidden machinery.

These may be explored later only when they preserve the core bootstrap goal.
