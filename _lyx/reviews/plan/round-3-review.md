---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/02-drift-records.md:37
    summary: >-
      halt_named_at and gate_halt_at carry no format, and the resume test compares them with the shed's second-truncated HistoryEntry.At, so a natural time.Time record never reads as a resume
    fix: >-
      In 02-drift-records.md, state that `halt_named_at`, `gate_halt_at` (and the other `*_at` fields the history test reads) are strings formatted `Deps.Now().UTC().Format(time.RFC3339)`, the format `shedengine`'s `nowRFC3339` gives `HistoryEntry.At` and `LastAt`, and that the history test compares both as parsed instants at second precision.
      In 11-drift-verify-producer.md, steps 3 and 7, say the same where the "at or after" test is named.
      Add a TestDriftVerify row whose fake `Now` returns a sub-second time and whose seeded history `At` is the same instant truncated to the second, asserting the entry is still read as the resume.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/11-drift-verify-producer.md:59
    summary: Step 7 clears gate_halt_at before the verify gate runs, so a gate interrupted mid-run leaves the next entry a non-resume that skips the gate and compares ungated code
    fix: >-
      In step 7 of 11-drift-verify-producer.md, keep both fields set while the dirty check and the told verify entry's gate run.
      Clear them in one `UpdateScope` call only once the dirty check passed and the gate either passed or was not due (`AttemptHead` absent or equal to HEAD); a failure rewrites both in one `UpdateScope` call, as the plan already says.
      Say that a resume which halts at step 3, 4, 5 or 6 leaves the fields set, so the following entry is still the resume from the gate halt and still runs the gate once.
      Add a TestDriftVerify row where the verify gate returns a context error on a resume with HEAD past the attempt head, then the next entry runs the gate again before any compare.
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/plan/14-drift-recipe-rows.md:79
    summary: The tracked dirty read is named only as through fabricengine, which leaves open whether the records checkout counts, where the row's own scope.json writes live
    fix: >-
      In 14-drift-recipe-rows.md, name the `driftDeps` dirty-paths seam as `fabricengine.SourceDirtyPaths(location)`, the code worktree's tracked dirt, not `Fabric.DirtyPaths`, which also lists the records checkout.
      In step 7 and the `dirty` gate entry of 11-drift-verify-producer.md, say the dirty tree is the code checkout's tracked paths and that records-side paths such as `_lyx/drift/scope.json` never count, since an agent's `lyx drift accept` and steps 2 and 3 write them before the check.
      Add a TestDriftVerify row with only a records-side path dirty, asserting no dirty Stuck.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/02-drift-records.md:38
    summary: gate_halt is written and cleared but never read, and card 2 says it lets the resumed entry tell the halt while card 11 says no code reads the opening texts
    fix: >-
      Drop `gate_halt` from the `ScopeRecord` field list in 02-drift-records.md and from steps 7 and 10 and the test paragraph of 11-drift-verify-producer.md, keeping `gate_halt_at`, the only field the resume test reads.
      Keep the opening-text package constants, which still open the Stuck reasons.
      Reword card 2's `gate_halt_at` line to say it is the time of the dirty-tree or test-gate Stuck the resumed entry compares the history against.
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/11-drift-verify-producer.md:78
    summary: No TestDriftVerify row reaches a resume from the row's own gate halt when HEAD equals the attempt head, the branch where D6 says the gate does not run
    fix: >-
      Add to the TestDriftVerify paragraph a row with `gate_halt_at` seeded, a last history entry that is a Stuck of this row at or after it, and `AttemptHead` equal to HEAD, so the dirty-tree halt is the common case.
      Assert the told verify entry is not called, both fields are cleared and the entry goes on to the compare.
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/plan/11-drift-verify-producer.md:48
    summary: Step 3 clears halt_named but leaves halt_named_at set
    fix: >-
      In step 3 of 11-drift-verify-producer.md, say the `ruled` marking clears `halt_named` and `halt_named_at` in the same `UpdateScope` call.
---

### [BLOCKING] Resume test compares unformatted record times with the shed's second-truncated history times

**Location:** `_lyx/plan/02-drift-records.md:37`, `_lyx/plan/11-drift-verify-producer.md:48`, `_lyx/plan/11-drift-verify-producer.md:57`

**Issue:** Steps 3 and 7 decide a resume by the last history entry being a Stuck of this row whose latest time is at or after `halt_named_at` or `gate_halt_at`.
`shedengine` stamps `HistoryEntry.At` and `LastAt` with `time.Now().UTC().Format(time.RFC3339)` (`nowRFC3339` in `internal/shedengine/run.go`), after the producer's `Call` has returned, so the value is truncated to the second.
Card 2 gives the two `*_at` fields no type or format, and `Deps.Now` is a `func() time.Time`.
A record field typed `time.Time` and set from `Now()` keeps its fraction.
The row writes the field milliseconds before the shed stamps the Stuck, usually in the same second, so the truncated `At` is earlier than the stored instant and the test reads false.
Under `full` the parent's resume is then never read as the resume from the decision halt: step 3 never marks `ruled`, step 4 halts again on the same `unruled` acceptance, and D6's "the parent's accept and resume then reaches Done" cannot happen.
`TestDriftVerify` runs on a fake `Now`, usually at whole seconds, so no named row catches it.
The plan is silent on the format, so a correct reading exists, but the natural one fails.

### [MEDIUM] Step 7 clears `gate_halt_at` before the gate runs

**Location:** `_lyx/plan/11-drift-verify-producer.md:59`

**Issue:** An entry that is the resume and passes the dirty check "clears both fields, then runs the told `verify` entry's gate once".
The gate is a test run that takes minutes.
If it ends in a context error, a transient stop or an interrupt, `shedengine` records no history entry (an empty outcome records nothing) and the status keeps the earlier Stuck as its last entry, but the fields are already gone.
The next entry then sees `gate_halt_at` unset, decides it is no resume, and goes on to the compare with the session's committed code never gated.
That breaks D6's order, in which the test gate runs before the compare so no fix goes uncompared or ungated.
Clearing only after the gate has passed (or was not due) removes the window, and a failed gate already rewrites both fields.
A resume that halts at steps 3 to 6 keeps the fields as the plan is written, and the following entry runs the gate once.
That is the right behavior, because the gate has not passed since, but the plan states "no other entry runs it".
Step 7 should say so.
Round 2 proposed clearing before the gate and its fixer applied that; nothing was disputed, and this finding rests on the interrupted-gate window, which that round did not weigh.

### [MEDIUM] The dirty read does not say which checkout it covers

**Location:** `_lyx/plan/14-drift-recipe-rows.md:79`

**Issue:** `_lyx/` is a link into the weft checkout, so `_lyx/drift/scope.json` and the logs are records-side files.
`driftDeps` builds `DirtyPaths` as "the tracked dirty paths through `fabricengine`".
`Fabric.DirtyPaths` lists both checkouts, while `SourceDirtyPaths` and `CodeDirtyPaths` read the code side only.
Steps 2, 3 and the agent's `lyx drift accept` rewrite `scope.json` before step 7's check and before the `dirty` gate entry runs.
If the seam includes the records side, the first of those leaves the tree dirty and the row halts on its own bookkeeping, or the gate asks the session to commit a path it cannot commit from the code worktree.
Which checkout counts is a choice the plan has to make, not leave to the implementer.

### [LOW] `gate_halt` has a writer and a clearer but no reader

**Location:** `_lyx/plan/02-drift-records.md:38`

**Issue:** Card 2 says `gate_halt` records the halt's opening text "so its resumed entry tells that halt".
Card 11 step 7 says the opening texts are "for the operator's reading; no code reads them back", and the resume test reads only `gate_halt_at` plus the history.
Step 7 treats the dirty-tree and test-gate halts alike, so no branch needs the text either.
A field with no reader, kept in `scope.json` and rewritten on every gate halt, is dead data, and the two cards disagree on its purpose.

### [LOW] No row reaches the equal-head side of the gate rule

**Location:** `_lyx/plan/11-drift-verify-producer.md:78`

**Issue:** D6's gate runs once, only when the recorded attempt head differs from HEAD.
The listed rows reach HEAD past the attempt head (failing and passing gate), an older or other-row history entry, and a first entry with no attempt head.
No row has the resume from the row's own halt with `AttemptHead` equal to HEAD, though a dirty-tree halt after an agent that committed nothing is the commonest way to be there.
That prescribed skip is reached by no named row.

### [NIT] `halt_named_at` is left set

**Location:** `_lyx/plan/11-drift-verify-producer.md:48`

**Issue:** Step 3 says `halt_named` is cleared, not `halt_named_at`.
The leftover time is harmless because the named list is empty, but a later reader sees a time with no list.

## Rejected

- The question whether the resume trigger rests on data a resumed entry can read: it does not rely on the run status's error text, which the step-3b resume write empties, nor on `HistoryEntry.Reason`.
  It reads `gate_halt_at` from `scope.json` and the status history, which a resume leaves intact (`lyx loom resume` appends no history entry; only a goto does), with `LastAt` covering the fold of repeated budget-exempt Stucks.
  Not a defect apart from the findings above.
- The three entry kinds against D6: a first entry reads `AttemptHead` found false and a reroute or goto entry reads a last history entry that is no Stuck of this row; both run no gate, and both are reached by the row that seeds `gate_halt_at` with HEAD past the attempt head.
  The resume from the row's own halt reads `gate_halt_at`, the history and `AttemptHead`, and is reached by the failing-gate and passing-gate rows.
  The equal-head side is the gap in the LOW finding above.
- Field names and card order: `gate_halt`, `gate_halt_at`, `halt_named` and `halt_named_at` appear under the same names in card 2 and card 11, and card 2 precedes card 11.
  Types are not given in card 2, which is covered by the BLOCKING finding.
