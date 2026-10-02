# Siml

**Siml (Simple Language)** is a systems programming language focused on making a usable compiler simple to implement on new operating systems, new instruction sets, and experimental architectures.

The project does **not** primarily aim to beat C in runtime performance or to minimize large-project build times. Its primary design constraint is that a bootstrap compiler must remain simple enough to implement in [Assam Core](https://github.com/tomiya7688/Assam).

## MVP direction

The initial implementation plan is:

1. Build a reference compiler in Go or Rust.
2. Build a minimal bootstrap compiler in Assam Core.
3. Keep the Assam Core implementation within an explicit complexity budget.
4. Implement the compiler in Siml itself.
5. Reach self-hosting: the Siml compiler can compile the Siml compiler.

See:

- [Design principles](docs/design-principles.md)
- [Open design decisions](docs/open-decisions.md)

Status: early design.
