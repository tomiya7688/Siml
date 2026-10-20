> English translation. The Japanese documents in the repository root and `docs/` are canonical.

# Pointer Semantics

Status: MVP draft decision  
Last updated: 2026-10-21

This document records the current Siml pointer decisions. The pointer and memory model is still incomplete; only the rules already decided are fixed here.

## Pointer arithmetic

Siml supports a deliberately small set of pointer arithmetic operations because they are directly useful for operating-system and low-level programming.

For a pointer `p: *T`, the following forms are supported:

```text
p + n
p - n
p1 - p2
```

No other arithmetic on pointers is implied by this feature.

In particular:

```text
n + p       -> compile error
p1 + p2     -> compile error
p * n       -> compile error
p / n       -> compile error
```

Restricting the valid forms keeps parsing, type checking, specification rules, and lowering small.

## Element-scaled offsets

`p + n` and `p - n` move by elements of `T`, not by raw bytes.

Conceptually:

```text
p + n  -> address(p) + n * sizeof(T)
p - n  -> address(p) - n * sizeof(T)
```

For example, if `p: *u32` and `u32` is four bytes wide:

```text
p + 1  -> four bytes after p
p + 2  -> eight bytes after p
```

Byte-oriented address traversal can use a `*u8` pointer.

The exact integer type or types accepted for `n` are still open and should be chosen to keep the compiler and specification simple.

## Pointer difference

Subtracting two pointers of the same pointer type produces their distance in elements, not bytes.

Conceptually, for `p1: *T` and `p2: *T`:

```text
p1 - p2 -> (address(p1) - address(p2)) / sizeof(T)
```

The result type is `isize`.

Example for `*u32`:

```text
p1 = 0x1010
p2 = 0x1000

p1 - p2 -> 4
```

The validity rules for subtracting pointers that do not refer to the same logical memory region are still open.

## No implicit array-to-pointer decay

An array value does not implicitly become a pointer.

This intentionally avoids C-style array-to-pointer decay and keeps array values and pointer values as distinct concepts.

Any operation that obtains a pointer to array storage must be explicit.

The exact syntax for taking the address of an array element or array storage is still open.

## No implicit pointer/integer conversion

Pointer arithmetic does not introduce implicit conversion between pointers and integers.

Pointer-to-integer and integer-to-pointer conversion, if supported, must be explicit.

The exact syntax and semantics of those conversions are still open.

## Design rationale

These rules are intended to preserve the operations needed for kernels, allocators, drivers, binary data traversal, and similar OS work while keeping the compiler smaller than a comparable C compiler.

The initial model deliberately avoids:

- commutative `integer + pointer` syntax;
- arbitrary arithmetic operators on pointers;
- implicit array decay;
- implicit pointer/integer conversion;
- C-style conversion rules around pointer expressions.

The remaining pointer and memory model should continue to prefer explicit local rules over hidden conversions or whole-program analysis.
