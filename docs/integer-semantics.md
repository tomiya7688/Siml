# Integer Semantics

Status: MVP draft decision  
Last updated: 2026-10-02

This document defines the current integer model for Siml.

The rules are intentionally small. They should be easy to explain, easy to implement in the Bitlang VM Assembly bootstrap compiler, and safer and more predictable than C where that does not require substantial compiler or runtime complexity.

## Integer types

The current MVP integer set is:

```text
i8   i16   i32   i64
u8   u16   u32   u64
isize usize
```

The fixed-width types have exactly the width written in their names.

Signed fixed-width integers use two's-complement representation.

`usize` and `isize` use the target pointer width.

## No implicit casts between typed values

Typed integer values do not implicitly convert to other integer types.

For example:

```text
u8 + u32   -> compile error
i32 + i64  -> compile error
u32 == u64 -> compile error
```

An explicit conversion is required when converting between integer types.

The exact syntax and semantics of explicit casts are specified separately.

## Integer literals

Integer literals are a limited convenience and are not general implicit casts.

Rules:

1. An integer literal does not initially have a fixed integer type.
2. If exactly one integer type is required by the immediate context, the literal is interpreted as that type if the value is representable.
3. If the value is not representable, compilation fails.
4. If no integer type is required by context, the literal defaults to `i32`.
5. This contextual literal rule must not grow into a general constraint-solving type inference system.

Examples:

```text
let a: u8 = 10;      // OK
let b: u8 = 300;     // compile error
let c = 10;          // i32

let x: u64 = 100;
let y = x + 1;       // literal 1 is u64

let z: u32 = 10;
let q = x + z;       // compile error: u64 + u32
```

If contextual literals cause disproportionate complexity in the low-level bootstrap compiler, this convenience may be removed.

## Addition, subtraction, and multiplication

`+`, `-`, and `*` use fixed-width wrapping semantics for both signed and unsigned integers.

For an N-bit integer, the result is the mathematical result modulo `2^N`.

Examples:

```text
u8(255) + 1 -> 0
i8(127) + 1 -> -128
```

Overflow in these operations is defined behavior and is not undefined behavior.

No mandatory overflow check is inserted for ordinary wrapping arithmetic.

## Division

Unsigned division uses ordinary integer division.

Signed division rounds toward zero.

Examples:

```text
 7 /  3 ->  2
-7 /  3 -> -2
 7 / -3 -> -2
-7 / -3 ->  2
```

Division by zero traps.

For a signed integer type, `MIN / -1` traps because the mathematical result is not representable in the same type.

## Remainder

`%` follows the same division model.

The identity

```text
a = (a / b) * b + (a % b)
```

holds for valid operands.

The remainder has the sign of the dividend for signed integers.

Examples:

```text
 7 %  3 ->  1
-7 %  3 -> -1
 7 % -3 ->  1
-7 % -3 -> -1
```

Remainder by zero traps.

For consistency with signed division handling, `MIN % -1` traps.

## Bitwise operations

The following bitwise operations are available on integers:

```text
&
|
^
~
```

Operands of binary bitwise operations must have exactly the same integer type.

The result has that same type.

## Shift operations

A shift count must be less than the bit width of the shifted integer.

For an N-bit value:

```text
0 <= shift_count < N
```

A statically known invalid shift count is a compile error.

A dynamically invalid shift count traps.

Left shift discards bits shifted beyond the integer width.

Unsigned right shift is logical.

Signed right shift is arithmetic.

Examples for `u32`:

```text
x << 31 -> valid
x << 32 -> invalid
```

The runtime cost and bootstrap implementation cost of dynamic shift validation must be measured. If this rule proves disproportionately expensive for the Bitlang VM Assembly compiler or target model, the rule may be reconsidered.

## Comparisons

Integer comparisons require operands of exactly the same type.

Comparison results are always `bool`.

Examples:

```text
u32 == u32 -> bool
u32 <  u32 -> bool
u32 == u64 -> compile error
```

Integers do not implicitly convert to `bool`.

## Design rationale

The integer model is intentionally based on a few local rules:

- widths are explicit;
- typed values do not implicitly change integer type;
- ordinary arithmetic wraps rather than invoking undefined behavior;
- exceptional arithmetic cases have explicit trap behavior;
- comparison yields `bool`;
- contextual typing is limited to literals.

The bootstrap compiler should be able to implement these rules with local type checks and small lowering rules rather than C-style promotion tables or global type inference.
