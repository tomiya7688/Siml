# Siml

**Siml (Simple Language)** is a systems programming language focused on making a usable compiler simple to implement on new operating systems, new instruction sets, and experimental architectures.

The project does **not** primarily aim to beat C in runtime performance or to minimize large-project build times. Its primary design constraint is that a bootstrap compiler must remain simple enough to implement in [Bitlang VM Assembly](https://github.com/tomiya7688/Bitlang-VM), whose low-level profile is based on [Assam Core](https://github.com/tomiya7688/Assam).

## MVP direction

The initial implementation plan is:

1. Build a reference compiler in Go or Rust.
2. Build a minimal bootstrap compiler in Bitlang VM Assembly.
3. Keep the low-level bootstrap implementation within an explicit complexity budget.
4. Use that implementation both as a simplicity test and as a readable reference for developers bringing Siml to a new architecture or environment.
5. Implement the compiler in Siml itself.
6. Reach self-hosting: the Siml compiler can compile the Siml compiler.

See:

- [Design principles](docs/design-principles.md)
- [Integer semantics](docs/integer-semantics.md)
- [Open design decisions](docs/open-decisions.md)

Status: early design.
