# Pointer and Memory Model

Status: Proposal for review
Last updated: 2026-10-12

This document proposes a small pointer and memory model for the Siml MVP. None
of the rules in the proposal section are final until accepted. The unresolved
section records choices that still need agreement.

## Goals

- Support operating-system code and direct access to memory-mapped hardware.
- Keep ordinary loads and stores explicit and easy to lower.
- Avoid mandatory alias analysis, bounds metadata, allocation, or a garbage
  collector.
- Make object layout and address width predictable for a target.

## Proposed MVP baseline

### Pointer values

- A pointer has one pointee type and an access qualifier, written `*T` for
  writable access or `*const T` for read-only access in this proposal.
- A pointer value is one target address and has the target's pointer width.
- `usize` has that same width, as specified by the integer model.
- `usize` can represent an address, but converting between integers and
  pointers requires an explicit conversion.
- A null pointer value is available for every pointer type. Converting null
  between pointer types is allowed. A direct pointer conversion may preserve
  or weaken access, but may not remove the read-only qualifier.
- Pointer equality and inequality are available for pointers with the same
  base pointee type, regardless of access qualifier. Relational pointer
  comparisons are not part of the MVP.

### Addressing and access

- `&place` obtains the address of an addressable variable or field.
- `*pointer` reads or writes according to the pointer's access qualifier.
  Dereferencing null traps.
- Pointer arithmetic is not part of the MVP. Indexing can be added as a
  separate construct with explicit array rules.
- A non-null pointer must designate storage large enough for a value of its
  pointee type. The compiler does not insert general bounds or validity checks.
- Loads and stores are ordinary memory operations. The language does not
  promise atomicity or ordering between concurrent agents.
- Integer and pointer values use their declared widths. Aggregate layout is
  specified separately; this proposal does not define struct padding.

### Aliasing and low-level access

- The MVP has no `restrict`-like promise and no type-based strict-aliasing
  optimization rule. A compiler must preserve observable loads and stores
  through pointers that may refer to the same storage.
- Targets must provide a way to perform ordinary reads and writes to
  memory-mapped regions. A volatile-like access mode is unresolved below; until
  specified, this proposal does not claim that ordinary accesses are suitable
  for device registers.
- Allocation is not required by the language or its runtime.

## Narrow decision proposal: null dereference

**Recommendation:** dereferencing a null pointer must trap before the load or
store takes place. If the pointer is statically known to be null, reject the
operation at compile time when that is straightforward; dynamic null values
must be checked at runtime. The target defines how a trap is reported or
handled, but execution must not continue as if the access succeeded.

This rule applies only to null. It does not require general checks for whether
a non-null pointer is mapped, in bounds, or otherwise valid. Those checks would
need separate decisions and may be too expensive for the MVP. The trap mechanism
and recovery policy remain part of the broader failure-policy decision.

## Still unresolved

These choices affect code generation or safety enough that the proposal does
not settle them:

1. **Pointer syntax and mutability:** whether to adopt the `*T`, `*const T`,
   `&place`, and `*pointer` spellings below, and the proposed conversion and
   place-mutability rules.
2. **Alignment:** whether to adopt the target-profile alignment table below,
   its values for each primitive type, and the failure behavior of a misaligned
   access.
3. **Address representation:** whether every initial target can expose its
   data pointers through the single flat address space proposed below, or
   whether segmented or capability pointers must be supported from the start.
4. **Pointer/integer casts:** whether to adopt the proposed conversions below,
   which target value represents null, and whether targets can reserve it.
5. **Aliasing across types:** whether to adopt the scalar byte-access proposal
   below, and whether all targets must have 8-bit addressable units.
6. **Device access:** whether to adopt the explicit volatile operations below,
   their ordering guarantees, and the target fence interface below.
7. **Unsafe boundary:** whether raw dereference and address conversion are
   always available or must appear inside an explicit unsafe construct.
8. **Array indexing:** whether to adopt the fixed-array-only rule below, its
   exact syntax, and whether dynamic bounds traps fit the MVP budget.
9. **Null dereference:** whether to adopt the trap behavior above and how a
   target reports or handles a trap.

## Narrow decision proposal: explicit unsafe blocks

**Recommendation:** do not require an `unsafe { ... }` block for raw pointer
operations in the MVP. Keep dereference and pointer/integer conversion explicit
in the syntax, document their preconditions, and define their invalid cases as
part of the failure policy.

Reasoning:

- An `unsafe` block does not make an unchecked address valid or add runtime
  checks. It only marks where the programmer accepts responsibility.
- Enforcing blocks adds parser and semantic state, while values obtained inside
  a block can still escape it and be dereferenced elsewhere.
- Raw memory access is a primary systems-programming operation for Siml, so
  requiring a marker around every use may add syntax without reducing the
  minimum compiler's implementation work.
- A later optional lint or a stricter language profile could require explicit
  unsafe regions without changing pointer representation or load/store rules.

This is a recommendation, not an accepted decision. If accepted, the remaining
work is to define the preconditions and invalid cases for dereference and
pointer/integer conversion; this section does not imply that those cases are
undefined.

## Narrow decision proposal: pointer syntax

**Recommendation:** use `*T` for a pointer to `T`, `&place` to take the address
of an addressable place, and prefix `*pointer` to access the pointed-to place.
For example, `**T` is a pointer to a pointer to `T`. A `*pointer` expression
can be read or written subject to the language's place-mutability rules.

This gives the type and expression forms a direct recursive-descent grammar.
It does not require a separate member-access arrow or pointer arithmetic
syntax. The spellings are a proposal; precedence details belong in the later
concrete grammar.

Pointee mutability and variable mutability are proposed separately below.
The pointer qualifier describes access through the pointer; the ordinary
variable rules still determine whether the pointer variable itself can be
reassigned.

## Narrow decision proposal: pointee mutability

**Recommendation:** use `*T` for writable access to a `T` pointee and `*const T`
for read-only access. The qualifier describes access through the pointer; it
does not decide whether the variable holding the pointer can be reassigned.
That remains governed by the ordinary variable mutability rules.

Dereferencing `*T` produces a readable and writable place, subject to the
ordinary place-mutability rules. Dereferencing `*const T` produces a readable,
non-writable place. Taking the address of a writable place with `&place`
produces `*T`; taking the address of a read-only place produces `*const T`.
Converting `*T` to `*const T` is implicit and preserves the address. A direct
conversion from `*const T` to `*T` is disallowed. The separate pointer/integer
cast proposal allows explicit conversion through `usize`; under that proposal,
converting the integer back to `*T` yields writable access. The read-only
qualifier is therefore a static restriction on that pointer value, not a
safety boundary.

The qualifier does not change pointer width or representation. Pointer
equality may compare pointers to the same base type regardless of this
qualifier. This is a static access restriction, not hardware protection: it
does not prevent another writable pointer from changing the same storage.
These rules are a proposal and depend on accepted variable and cast rules.

## Narrow decision proposal: typed access alignment

**Recommendation:** require each target profile to give an `alignof(T)` value
for every primitive integer and data pointer type. Do not derive alignment
from a type's size or bit width; the target table is authoritative. Aggregate
and `bool` alignment remain with their separate type and layout decisions.
`alignof(T)` is a compile-time property. A pointer used to access a `T` value
must satisfy that alignment, and taking the address of a valid `T` place
produces a suitably aligned pointer. The compiler does not insert a dynamic
alignment check.

This keeps ordinary access as a direct load/store and gives the compiler a
simple precondition. An access whose alignment cannot be established from the
program violates that precondition; whether it must trap or has another defined
failure result belongs to the still-open failure policy. This proposal does
not promise that every target traps on a misaligned access.

This recommendation is not final. In particular, byte-oriented access to
packed data or devices would need a separate explicit operation or rule.

## Narrow decision proposal: flat data pointers

**Recommendation:** represent every MVP data pointer as one scalar address in a
single flat target data-address space. Every data pointer has the target pointer
width, matching the existing rule that `usize` uses that width. `usize` can
represent every usable data address except the target-reserved null pattern.
The address may name virtual or physical memory according to the target; this
rule does not require physical addresses. The compiler does not attach bounds,
ownership, or provenance metadata to a pointer.

This keeps pointer values scalar and supports a small backend representation.
It excludes segmented pointers, separate non-flat data address spaces, and
capability pointers from the MVP unless the target ABI exposes usable data
pointers through the required flat integer address representation. Such
targets could be handled by a later extension or a distinct target profile.

This recommendation fixes representation only. Pointer conversions and null
encoding are discussed in the following proposals and remain unaccepted.

## Narrow decision proposal: pointer conversions

**Recommendation:** permit explicit conversions between a data pointer and
`usize`, plus explicit conversions between pointer types. Do not convert any
other integer type directly to a pointer. An integer must first be explicitly
converted to `usize` under the general integer-cast rules. Pointer-to-`usize`
and `usize`-to-pointer conversions preserve all pointer-width bits. A direct
conversion between pointer types may change the base pointee type, but cannot
strengthen access: `*T` may convert to `*const U`, while `*const T` cannot
convert directly to `*U`. Allow an implicit `*T` to `*const T` conversion.

Under the flat-address proposal, these conversions preserve the pointer-width
address bits. Converting a `usize` to a pointer does not check whether that
address is mapped or valid for dereference. Such a pointer may only be
dereferenced when it meets the storage, type, and alignment preconditions.
Converting between pointer types does not check the address or change its
bits. Converting `*const T` to `usize` and then explicitly converting that
integer to `*U` produces writable access; this is allowed by the proposed
integer-cast rule and does not validate the address or make it safe to access.

Pointer/integer conversions use the target's null representation described in
the following proposal.

This is a proposal, not a final rule. It fixes the round-trip behavior only
under the flat-address and reserved-null assumptions; it does not establish
that an arbitrary non-null address is valid to access.

## Narrow decision proposal: null representation

**Recommendation:** require each flat-address target profile to reserve one
`usize` bit pattern for null. The target should choose a value that does not
name a usable non-null data address; the language does not require the value to
be zero.

Pointer-to-`usize` conversion preserves all bits, including the reserved null
pattern, and converting that pattern back to any pointer type produces null.
Every other `usize` bit pattern converts to a non-null pointer containing those
address bits, whether or not the address is mapped. This keeps null checks and
conversions to simple scalar comparisons while letting each target choose an
unmapped sentinel when available.

The cost is explicit: the reserved value cannot also designate a non-null data
address. A target with no spare representation cannot use nullable flat
pointers under this profile; it would need a different pointer profile or a
later representation extension. This proposal does not settle whether such a
target must be supported by the MVP.

This remains a proposal. The exact target null value belongs in the target
contract and is not fixed here.

## Narrow decision proposal: scalar byte representation and aliasing

**Recommendation:** require MVP targets to provide 8-bit addressable bytes.
Every integer and data pointer occupies its width divided by eight consecutive
bytes. The target contract specifies little-endian or big-endian byte order for
multi-byte values. The pointer width must therefore be a multiple of eight
bits.

Loads and stores through integer or pointer types access the corresponding
bytes in memory. Overlapping accesses through different integer or pointer
types observe the same bytes: a store through one type changes the result of a
later overlapping load through another type. A store changes only the bytes
occupied by its value; it leaves neighboring bytes unchanged. For example, if
a `u16` containing `0x1234` is stored and the first byte is loaded through a
`u8` pointer, the result is `0x34` on a little-endian target and `0x12` on a
big-endian target. This makes the proposal against type-based strict-aliasing
assumptions concrete. Each access still needs enough storage and the alignment
required by its accessed type.

The rule covers integer and pointer representations only. It does not define
the representation or byte access rules for `bool`, structs, unions, or padding;
those depend on the primitive and aggregate decisions. Targets with non-8-bit
addressable units would not meet this MVP profile unless they can expose a
compatible 8-bit addressable view.

This is a proposal, not an accepted restriction on targets. It trades some
architecture coverage for direct byte loads/stores and a small aliasing rule.

## Narrow decision proposal: volatile device access

**Recommendation:** add two compiler-recognized operations, written here as
`volatile_load(pointer)` and `volatile_store(pointer, value)`. The load returns
the pointed-to type; the store requires the value to have that same type.
These names are illustrative and do not settle the final syntax.

Each operation performs exactly one memory access of the pointee width. A
volatile access is observable: the compiler cannot remove, duplicate, merge, or
reorder it with another volatile access. The compiler also cannot move an
ordinary memory access across a volatile operation. These rules preserve the
written order of accesses without making every pointer or variable volatile.

Volatile operations do not make an access atomic, synchronize threads, or
promise that the processor or device observes accesses in that order. Targets
that need hardware ordering must provide a separate target-specific memory
fence. The ordinary pointer validity and alignment preconditions still apply;
the compiler does not check whether the address is mapped to a device.

This keeps device access explicit and local to two operations. It remains a
proposal; the exact ordering contract and target fence interface need review.

## Narrow decision proposal: target memory fence

**Recommendation:** keep hardware memory fences out of the portable core
language. A target that needs one exposes a compiler-recognized target
intrinsic, named here `target_memory_fence()` for illustration.

The intrinsic acts as a full barrier for memory operations: the compiler may
not move reads or writes across it, and the backend emits the target barrier
needed to order accesses for the observer domain documented by that target.
The target contract must say whether that domain includes devices as well as
other processors. Targets that need no such barrier may omit the intrinsic or
define a no-op where appropriate.

A fence is not an atomic operation, lock, or language-level thread
synchronization primitive. It does not make a compound read/modify/write safe.
This keeps architecture-specific ordering out of the portable language while
giving OS code a defined target hook. The proposal and exact target contract
remain open.

## Narrow decision proposal: fixed-array indexing

**Recommendation:** allow `array[index]` only when the operand is a fixed-size
array with a compile-time element count. Do not apply indexing to raw pointers
and do not implicitly convert arrays to pointers.

The index has type `usize`. A statically known index outside the array length
is a compile error; a dynamically out-of-range index traps. A valid access
addresses the `index`th contiguous element and produces an addressable place of
the element type. Lowering may calculate that address internally, but this does
not add pointer arithmetic to the source language.

The dynamic check requires a comparison and branch for each access unless the
compiler can prove the index is in range. That cost should be measured against
the bootstrap simplicity goal when a reference compiler exists. This proposal
does not settle array type syntax, zero-length arrays, or the trap mechanism.

## Decisions requested for this draft

The most consequential baseline choices to accept or revise are:

- typed pointers with target pointer width;
- pointer syntax using `*T`, `*const T`, `&place`, and `*pointer`; direct
  pointer conversions only preserve or weaken access, while an explicit
  integer round trip can produce a writable pointer;
- one flat data-address space with scalar pointers and no hidden metadata;
- explicit pointer/integer conversions;
- pointer conversions preserve address bits, using a target-reserved null
  value that need not be zero;
- null pointers, with null dereference trapping before memory access;
- no pointer arithmetic in the MVP;
- fixed-size array indexing only, with `usize` indices and dynamic bounds
  traps;
- no general pointer-validity checks beyond fixed-array bounds;
- no type-based strict-aliasing assumption;
- integer and pointer scalar storage is byte-addressable in 8-bit units, with
  target-specified byte order;
- target-profile `alignof(T)` values for primitive integer and data pointer
  types, with access alignment as a precondition and no mandatory dynamic
  check;
- defer exact alignment values, `bool` and aggregate representations, and
  the final volatile access and target-fence contracts to explicit follow-up
  decisions.

Once this baseline is agreed, the unresolved list can be split into separate
small decisions and the accepted rules moved into the MVP specification.

