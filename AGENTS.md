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
- Pointer decisions: `docs/pointer-semantics.md` records accepted rules.
- Pointer and memory proposals: `docs/pointer-memory-model.md`. Its contents
  are discussion for review, not accepted language rules. Japanese documents
  are canonical; `docs/en/` contains English translations.

Do not read all design documents by default. Use `rg` to locate the exact topic
and inspect the relevant section. When changing semantics, update the owning
specification and its decision status together; keep proposals visibly separate
from accepted rules.

## Project stage

Siml is in early design; no compiler implementation is present yet. CI checks
UTF-8 Markdown and JSON and the initial formatter coverage. Install the pinned
formatter with `npm ci --ignore-scripts`, run `npm run format -- <files>` and
`npm run format:check -- <files>` for changed files, and use
`npm run format:check:ci` for the complete CI formatter check. For documentation
edits also run `git diff --check` and review the diff. Do not describe
proposals as implemented behavior.

Do not add indexes, context packs, or helper tools for this small document set
unless repeated work shows that the saved search effort will exceed their
maintenance cost.
