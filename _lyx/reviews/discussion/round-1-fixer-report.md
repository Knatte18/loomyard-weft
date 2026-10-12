# Fixer report — discussion, round 1

## Changes

- F1 (MEDIUM): fixed as proposed.
  D1 now requires a kept value to survive every write path byte-for-byte, `ApplyPatch` included, naming the requirement and not the mechanism.
  The first Acceptance criterion's unknown scalar is an integer above 2^53 (9007199254740993), pinned through the upsert and batch-upsert rows.
- F2 (MEDIUM): fixed as proposed.
  D4, the `internal/boardcli/doc.go` Scope bullet, the print criterion and the last criterion's `boardcli/doc.go` clause all name `get`, `list-full` and the task envelope of `upsert`, `promote` and `merge`.
  Support log Q6 revised to match.
- F3 (MEDIUM): fixed, taking both bounds the fix offered rather than one.
  D7 retries only on the directory-not-empty symptom and reports any other error after the first attempt; the one-second budget is shared across one `Root.Sweep` call.
  The symptom rule alone still lets a sweep with several racing directories cost a second each, and the shared budget alone still makes a persistent failure cost every `newRun` a second; together the Bound states one `newRun`'s worst case as one second, and a non-race failure as one attempt per directory.
  Acceptance criteria gained a no-fresh-budget clause and a one-attempt row for other errors; the auto-mode assumption and support log Q7 were revised to match.
- F4 (LOW): fixed as proposed in D4's Bound.
- F5 (LOW): fixed as proposed.
  D7 names a retry clock-and-sleep pair separate from the creation-grace `now`, defaulting to `time.Now` and `time.Sleep`, with the injected sleep advancing the injected clock in tests; the plan-writer note now warns that fixtures freeze `now` with `fixedClock`.
- F6 (NIT): fixed as proposed; D7's premise moved into a Rationale line after the Decision text.

## Disputed

None.

## Deferred

None.
