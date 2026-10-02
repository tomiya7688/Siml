# Open Design Decisions

Status: Draft  
Last updated: 2026-10-02

This document lists the decisions that must be made while designing Siml.

The purpose is not to decide every feature immediately. It is to make the design surface explicit and to keep decisions ordered by their effect on the bootstrap compiler.

## Decision rule

When multiple designs are reasonable, prefer the design that:

1. preserves the mandatory requirement that the minimum Siml compiler remain simpler than a comparable minimum C compiler for freestanding / OS work;
2. keeps the Bitlang VM Assembly bootstrap compiler smaller and easier to understand;
3. keeps runtime requirements explicit and small;
4. produces reasonably direct low-level code without sophisticated optimization;
5. gives source programs more predictable behavior than C where that can be achieved cheaply;
6. remains expressive enough to write the Siml compiler and operating-system code.

Syntax convenience is secondary to these constraints.

---

# P0: Decisions required for the first compiler

These decisions define the minimum language that the Go/Rust and Bitlang VM Assembly bootstrap compilers must share.

## 1. Compilation model

Decide:

- What is the smallest independently compilable unit?
- Does one invocation compile exactly one source file or one module?
- Is Bitlang VM Assembly the required MVP output format?
- Are symbols emitted in a form suitable for later linking?
- How are constants and static data represented in the output?

Current direction:

- Bitlang VM Assembly is the canonical MVP lowering target. Its strict low-level profile is based on Assam Core.
- Direct native-code backends may exist later but must not be required to define the language.

## 2. Primitive types

Decide the exact primitive type set.

Questions include:

- Which fixed-width signed integers exist?
- Which fixed-width unsigned integers exist?
- Is there a native machine-word integer type?
- Is `bool` a distinct type?
- Is there a language-level character type?
- Are floating-point types part of the MVP?
- How are integer literals typed?

The type set should not require complicated promotion rules.

## 3. Integer semantics

Every integer operation needs deterministic rules.

Decide:

- overflow behavior;
- signed and unsigned division;
- division by zero;
- modulo behavior;
- left and right shift behavior;
- invalid shift counts;
- comparison semantics;
- whether arithmetic operands must have exactly matching types;
- which conversions are implicit, if any.

A major goal is to avoid C-style promotion and conversion complexity unless it provides clear value.

## 4. Pointer and memory model

This is central for OS development.

Decide:

- pointer syntax and pointer types;
- address-of and dereference operations;
- whether pointer arithmetic exists;
- whether a null pointer value is defined;
- alignment requirements;
- behavior of misaligned access;
- load/store widths;
- aliasing rules;
- representation of addresses;
- casts between integers and pointers;
- support for memory-mapped I/O;
- whether a `volatile`-like concept is required.

The rules must be implementable without a sophisticated alias analysis.

## 5. Variables and mutability

Decide:

- declaration syntax;
- whether variables are mutable by default;
- whether initialization is mandatory;
- whether uninitialized storage can be explicitly requested for low-level use;
- local variable storage rules;
- global/static variables;
- constants.

The bootstrap compiler should not require control-flow-heavy definite-assignment analysis unless there is a strong reason.

## 6. Expressions and evaluation order

Decide:

- operator set;
- precedence;
- associativity;
- evaluation order;
- assignment as statement or expression;
- short-circuit boolean behavior;
- side effects inside expressions.

Evaluation order should be explicit and deterministic.

## 7. Control flow

Decide the MVP control-flow constructs.

Candidates include:

- `if` / `else`;
- `while`;
- infinite loop;
- `break`;
- `continue`;
- `return`;
- `goto` and labels;
- a basic `for` form.

Every construct should lower mechanically to explicit branches.

## 8. Functions

Decide:

- function declaration syntax;
- argument passing model;
- return values;
- recursion;
- function pointers;
- multiple return values, if any;
- variadic functions;
- external functions;
- forward declarations;
- visibility.

The language-level function model must remain compatible with a simple explicit calling convention.

## 9. Aggregate data

Decide the minimum aggregate types.

Likely areas:

- fixed-size arrays;
- structs/records;
- field layout;
- padding and alignment;
- anonymous aggregates;
- unions;
- tagged unions/enums.

For OS work, memory layout must be predictable.

## 10. Casts and conversions

Decide exactly which conversions are:

- automatic;
- explicit but safe;
- explicit and low-level;
- forbidden.

A small conversion table is preferable to a large set of contextual promotion rules.

## 11. Failure, trap, and undefined behavior policy

For each invalid operation, decide whether it:

- is rejected at compile time;
- has defined wrapping or other semantics;
- traps at runtime;
- has implementation-defined behavior;
- is explicitly undefined.

Important cases include:

- integer overflow;
- division by zero;
- invalid shifts;
- null dereference;
- invalid pointer access;
- out-of-bounds array access;
- invalid casts;
- unreachable states.

Siml should avoid undefined behavior when a cheap deterministic rule is available, but it must not require expensive runtime checks merely to eliminate every possible undefined operation.

## 12. Minimum parser strategy

The grammar should be intentionally compatible with a small hand-written parser.

Decide:

- tokenization rules;
- comment syntax;
- statement termination;
- block syntax;
- declaration grammar;
- whether parsing requires arbitrary backtracking;
- whether contextual keywords exist.

A recursive-descent or similarly straightforward parser should be sufficient.

---

# P1: Decisions required for useful OS-scale programs

## 13. Modules and imports

Decide:

- mapping between files and modules;
- import syntax;
- symbol visibility;
- cyclic dependencies;
- name qualification;
- whether interfaces/header-like summaries exist;
- how much imported source a compiler must read.

The first design should favor compiler simplicity over advanced incremental compilation.

## 14. Separate compilation and linking

Decide:

- what metadata is emitted between compilation units;
- symbol naming;
- relocation representation;
- how static data is referenced;
- how Bitlang VM Assembly output from multiple units is combined;
- whether a linker is considered part of Siml or an external tool.

## 15. ABI boundary

Distinguish the language semantics from a target ABI.

Decide:

- Siml-to-Siml calling convention requirements;
- external ABI annotations;
- C ABI interoperability, if supported;
- register/stack responsibilities;
- struct argument and return handling;
- stack alignment;
- platform-specific escape hatches.

A new architecture should not need a large ABI implementation before simple Siml code can run.

## 16. Runtime contract

Write down the complete set of services a conforming minimal environment must provide.

Candidates include:

- none beyond raw program entry;
- trap handling;
- memory copy/set helpers;
- integer helpers for operations unsupported by the target;
- optional allocator interface;
- startup/termination hooks.

The required runtime should be small enough to audit as part of a new platform port.

## 17. Platform and unsafe interfaces

OS code needs explicit access to machine facilities.

Decide how Siml represents:

- inline or external assembly;
- raw Bitlang VM Assembly / Assam Core blocks, if any;
- special registers;
- interrupts;
- memory barriers;
- atomic operations;
- system calls;
- architecture-specific intrinsics.

These should not contaminate the portable core more than necessary.

---

# P2: Decisions that can wait until the bootstrap core works

Potential later features include:

- richer enums/tagged unions;
- slices;
- generics;
- methods;
- interfaces/traits;
- compile-time evaluation;
- macros;
- richer standard-library abstractions;
- optional safety checks;
- convenience iteration syntax;
- advanced optimization annotations.

Each of these must pass the feature-admission questions in the design principles before becoming part of the core language.

---

# Compiler complexity budget

The MVP requires a numerical or otherwise testable complexity budget for the Bitlang VM Assembly bootstrap compiler.

The exact limits are not yet decided, but the project should track at least:

- Bitlang VM Assembly source/instruction count;
- number of compiler passes;
- number of compiler functions/routines;
- number and kind of internal data structures;
- amount of required dynamic allocation;
- number of mandatory runtime/helper routines;
- maximum semantic state that must survive between passes;
- number of target-specific special cases in the language frontend.

The budget should measure architectural complexity, not reward code golfing.

A shorter implementation that is difficult to understand is not automatically simpler.

## Candidate acceptance model

A future MVP gate could require all of the following:

- hand-written lexer;
- hand-written parser;
- straightforward symbol table;
- local and deterministic type checking;
- no mandatory optimizer;
- no mandatory SSA;
- no garbage collector;
- bounded and documented helper runtime;
- direct or very small-IR lowering to Bitlang VM Assembly.

The numerical limits should be chosen only after the first Go/Rust implementation provides enough information to estimate what is realistic.

---

# Self-hosting acceptance test

Self-hosting should have an explicit test rather than a symbolic milestone.

At minimum:

```text
reference compiler
    -> compiles Siml compiler source
    -> produces self-hosted compiler A

compiler A
    -> compiles the same Siml compiler source
    -> produces compiler B
```

Compiler B must successfully compile representative Siml programs.

Whether A and B must be byte-for-byte reproducible is a separate decision.

---

# Recommended decision order

The next design work should proceed roughly in this order:

1. primitive types and integer semantics;
2. pointer/memory model;
3. expressions and evaluation order;
4. variables and aggregates;
5. control flow;
6. functions and the minimum calling model;
7. failure/trap/undefined-behavior policy;
8. concrete grammar;
9. Bitlang VM Assembly lowering rules;
10. module and separate-compilation model;
11. runtime/ABI boundary;
12. complexity-budget numbers;
13. self-hosted compiler design.

This order tries to make the first executable compiler possible before solving large-project ergonomics.
