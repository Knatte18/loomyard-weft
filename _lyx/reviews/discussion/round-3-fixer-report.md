# Round 3 fixer report — discussion

## Fixed

- **F1** (MEDIUM, design): I confirmed the premise against the code and the spec.
  `checkHandleConsistency` in `internal/planparser/validate.go` reports `handle-collision` for a second claim on one handle.
  The loom plan spec's `handle-malformed` row makes a handle's unit the package directory, never a `.go` file, so a Create per file can't be written.
  - D6 in `_lyx/discussion/decision-record.md` now names this way forward for a partitioned Rename old side: a Delete of the old member plus one Create declaration of the new member's handle, on a card that edits each of the old member's files, or an Edit of each file's body.
    It also states that one declaration head serves every file even when the per-file signatures differ.
    The head only computes the new glyph, and partitioned declarations share one glyph.
    D6 says outright that a Create per file is never the way forward, because it trips `handle-collision`.
  - The matching acceptance criterion now pins that wording.
  - `_lyx/discussion/support-log.md`'s Question ledger gains row 16, which records the decision.

## Disputed

None.

## Deferred

None.
