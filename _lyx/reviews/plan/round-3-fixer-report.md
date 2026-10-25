# Fixer report — plan, round 3

## Changes

- **F1** (BLOCKING), fixed as proposed, narrowed after the self-check.
  `02-drift-records.md` states that `halt_named_at` and `gate_halt_at` are strings `Drift-Verify` formats `Deps.Now().UTC().Format(time.RFC3339)`, the format `nowRFC3339` gives `HistoryEntry.At` and `LastAt` (`internal/shedengine/run.go:593`), and that the history test parses both and compares them as instants at second precision.
  Steps 3 and 7 of `11-drift-verify-producer.md` say the same where "at or after" is named.
  `TestDriftVerify` gains a row with a sub-second fake `Now` and a second-truncated seeded `At`, read as the resume for both fields.
- **F2** (MEDIUM), fixed as proposed, plus one change from the self-check.
  Step 7 keeps `gate_halt_at` set while the dirty check and the verify gate run.
  It clears the field in one `UpdateScope` call only once the dirty check passed and the gate passed or was not due.
  A resume that halts at step 4, 5 or 6 leaves the field set.
  The self-check also changed what a halt writes: it writes `gate_halt_at` only when the field is unset, so a resume's failing gate keeps the time it read (see the fix-proposal line under Self-check).
  `TestDriftVerify` gains the context-error row, in which the next entry runs the gate again before any compare.
- **F4** (LOW), fixed as proposed.
  `gate_halt` is dropped from card 2's field list and from steps 7 and 10 and the test paragraph of card 11.
  Card 2's `gate_halt_at` line now says it is the time of the dirty-tree or test-gate Stuck that the resumed entry compares the history against.
  The opening-text constants stay.
- **F5** (LOW), fixed as proposed.
  `TestDriftVerify` gains a row: a resume from the row's own dirty-tree halt with `AttemptHead` equal to HEAD does not call the verify entry, clears `gate_halt_at` and goes on to the compare.
- **F6** (NIT), fixed as proposed.
  Step 3 clears `halt_named` and `halt_named_at` in the same `UpdateScope` call.
- **Parent decision** (from `lyx loom decision add`, "Drift-Verify's dirty check reads the code checkout's tracked paths only"), applied.
  In `14-drift-recipe-rows.md`, `driftDeps`' dirty-paths seam is `fabricengine.SourceDirtyPaths(location)`, never `Fabric.DirtyPaths`.
  Step 7 and the `dirty` gate entry of card 11 say that only the code checkout's tracked paths count and records-side paths never do.
  `TestDriftVerify` gains a row in which only a records-side path is dirty, with no dirty Stuck.

## Self-check

- fix-claims, LOW: step 7 named a halt at step 3, which cannot happen. Duplicate of fix-proposal's first LOW. Fixed as proposed: the text now reads "step 4, 5 or 6".
- fix-proposal, LOW: same finding as fix-claims' LOW, merged into it.
- fix-proposal, LOW: a failing resume that rewrote `gate_halt_at`, followed by an interrupt before the shed persisted the Stuck, made the next entry a non-resume that skips the gate. Fixed as proposed:
  - each step-7 and step-10 halt writes the field only when it is unset, so a resume keeps its time;
  - the failing-gate test row now asserts the seeded time is kept.
- fix-affected-set, MEDIUM: card 2's format sentence covered every `*_at` field, including fields card 7's verbs write without `Deps.Now`. Fixed as proposed: the sentence is narrowed to `halt_named_at` and `gate_halt_at`, which `Drift-Verify` writes.
- fix-consistency, MEDIUM: same finding as fix-affected-set's MEDIUM, merged into it.

## Disputed

None.

## Deferred

None.
