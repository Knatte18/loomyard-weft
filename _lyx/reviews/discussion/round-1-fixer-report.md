# Fixer report — discussion, round 1

I checked every finding against the code before fixing it.
All nine hold, and all nine are fixed in `_lyx/discussion/decision-record.md`.
`support-log.md` needed no change.

## Fixed

- **F1 (BLOCKING):** D8's pre-rename compare now uses the bytes `WriteCarryOver` read at its start, which matches `writeCarryOver`'s `prior`.
  A5 and the acceptance criterion now use the same wording.
- **F2 (MEDIUM):** D8's bound now says the checks catch a lost update in most interleavings, not all.
  R2 now names both open windows: a write landing between the compare and the rename, and an in-place read-modify-write (`AppendDecision` uses `os.WriteFile`, not a rename) that writes after the post-rename re-read.
- **F3 (MEDIUM):** D4 now clears the marker at both seed sites, the pre-run and pre-step hooks, through one `battenshed` export.
  Scope In, the acceptance criterion (fresh run via `lyx batten run` or `lyx batten step`) and the plan-writer note follow.
- **F4 (MEDIUM):** D4's bound is now conditional: an open window loses its marker only when the durable batten status vanishes while the scratch marker survives.
  The new risk R4 records that case.
- **F5 (MEDIUM):** D5 now says a value that does not decode as an integer is left untouched with no report line, so reconcile still runs for every module and the loader rejects the value as it does today.
  The acceptance criterion now covers that case.
- **F6 (MEDIUM):** D5 no longer claims byte identity.
  Every other key, value and comment is carried through by yaml node editing, and the applied write is reconcile's template-shaped merge.
  The acceptance criterion now compares the result with reconcile's output for the same input with the key already at the floor.
- **F7 (MEDIUM):** D6 now gives `retire-legacy` a `legacy_files` field in both modes.
  The bound now reads "today's behaviour plus `applied` and that report", and the acceptance criterion says the same.
- **F8 (LOW):** Dropped "the spec's own back-link" from D2.
  `contracts/specs/final-summary-spec.md` never names its own file.
- **F9 (NIT):** Deleted the sentence in D2's Rationale that restated A1.

## Disputed

None.

## Deferred

None.
