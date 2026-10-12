# AI development entry

Keep task context narrow. This file is the always-loaded project guide; open
only the source documents needed for the current change.

## Workflow

- State the task goal and acceptance check before exploring.
- Search filenames and symbols first; read matching sections rather than every
  document. Stop when the required evidence is sufficient.
- Keep unrelated edits out of the task. Treat summaries as pointers back to the
  source documents, not as replacements for them.
- Run the smallest relevant validation and report what remains unverified.

## Source routing

- Project purpose and constraints: `README.md`, then `docs/design-principles.md`
  when changing project direction.
- Integer rules: `docs/integer-semantics.md`.
- Accepted decisions and remaining design questions: `docs/open-decisions.md`.
- Pointer and memory work: `docs/pointer-memory-model.md`. Its contents are
  proposals for review, not accepted language rules.

Do not read all design documents by default. Use `rg` to locate the exact topic
and inspect the relevant section. When changing semantics, update the owning
specification and its decision status together; keep proposals visibly separate
from accepted rules.

## Project stage

Siml is in early design; no compiler implementation or automated test suite is
present yet. For documentation-only edits, use `git diff --check` and review the
resulting diff. Do not describe design proposals as implemented behavior.

Do not add indexes, context packs, or helper tools for this small document set
unless repeated work shows that the saved search effort will exceed their
maintenance cost.

