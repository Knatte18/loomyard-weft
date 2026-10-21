---
round: 1
focus:
  - "internal/planglyph/donecheck.go, the new `.scratch/` skip: does the prefix test run on the same repository-root-relative resolve key for plain paths and plan: handles (`resolveKeyFor`, a `#` suffix, a leading `./`), so every root `.scratch/` Create target the board entry describes is skipped and no `sub/.scratch/` target is?"
  - "The skip's reach against the board entry: only `Create` group targets are exempt, and `Delete` targets and `Rename` sides keep their checks; does anything downstream of `DoneChecks` (target lookup, `addTarget`, verdicts) still assume the skipped entry exists?"
  - "Docs landing with the change: the `DoneChecks` doc comment, `internal/planglyph/doc.go` and `contracts/specs/loom-plan-spec.md` state the same bounded exemption, and the plan-spec wording does not license a planner to rely on it for a path outside the root `.scratch/`."
  - "`TestDoneChecks_Verdicts` rows in `internal/planglyph/donecheck_resolve_test.go`: do the two new rows cover the skip and the `sub/.scratch/` boundary without duplicating an existing row, and is the change description's claim of what they cover borne out?"
---

Seed focus from the diff, the change description and the board entry; the reviewer still judges the whole committed diff.
