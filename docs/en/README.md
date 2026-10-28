> English translation. The Japanese documents in the repository root and `docs/` are canonical.

# Siml

**Siml (Simple Language)** is a systems programming language focused on making a usable compiler simple to implement on new operating systems, new instruction sets, and experimental architectures.

The project does **not** primarily aim to beat C in runtime performance or to minimize large-project build times. Its primary design constraint is that a bootstrap compiler must remain simple enough to implement in [Bitlang VM Assembly](https://github.com/tomiya7688/Bitlang-VM), whose low-level profile is based on [Assam Core](https://github.com/tomiya7688/Assam).

Siml aims to be sufficient for implementing an operating system end to end, from low-level machine-facing code to higher-level userspace software, without requiring another systems language. A correct non-optimizing compiler is a complete Siml compiler; optimization is optional and should remain separable from the minimum implementation.

## MVP direction

The initial implementation plan is:

1. Build a reference compiler in Go or Rust.
2. Build a minimal bootstrap compiler in Bitlang VM Assembly.
3. Keep the low-level bootstrap implementation within an explicit complexity budget.
4. Use that implementation both as a simplicity test and as a readable reference for developers bringing Siml to a new architecture or environment.
5. Implement the compiler in Siml itself.
6. Reach self-hosting: the Siml compiler can compile the Siml compiler.

See:

- [Design principles](design-principles.md)
- [Integer semantics](integer-semantics.md)
- [Pointer semantics](pointer-semantics.md)
- [Open design decisions](open-decisions.md)

## Documentation formatting

Run these commands from the repository root with Node.js 24 and npm. First run `npm ci --ignore-scripts` to install the pinned Prettier 3.6.2. Pass the files being changed to the commands:

```sh
npm run format -- docs/pointer-memory-model.md docs/open-decisions.md docs/en/open-decisions.md
npm run format:check -- docs/pointer-memory-model.md docs/open-decisions.md docs/en/open-decisions.md
npm run format:check:ci
```

`format:check:ci` checks the complete list of documents and formatter configuration used by CI. When changing other documents, pass those files to `format` and `format:check`.

Status: early design.
