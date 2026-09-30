---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/02-step-fold-wiring.md:34
    summary: step_fold_test.go never lands a no-verdict write right after a fold, so the carry-forward side of the new persistTransient discriminator and D4's pause-spanning fold go untested
  - id: F2
    severity: LOW
    location: _lyx/plan/02-step-fold-wiring.md:40
    summary: the exempt Stuck, Awaiting, exempt Stuck case folds nothing before the Awaiting, unlike the acceptance criterion, and does not say the third Step re-calls the producer through the step-3b resume write
  - id: F3
    severity: LOW
    location: _lyx/plan/02-step-fold-wiring.md:47
    summary: card 2's ImpactSummary omits the readers of history length, the one observable blast radius of a fold (D10)
  - id: F4
    severity: LOW
    location: _lyx/plan/02-step-fold-wiring.md:3
    summary: battencli's maxStatusHistoryEntries doc comment still claims Run-Shed appends one entry per awaiting and driver-exit poll, and no card corrects it
  - id: F5
    severity: NIT
    location: _lyx/plan/04-awaiting-resume-pending.md:40
    summary: card 4 leaves implicit that the third Call's child history length equals the recorded length, which is what makes it the D9 regardless-of-length case
---

### [MEDIUM] No no-verdict write after a fold in step_fold_test.go

**Location:** `_lyx/plan/02-step-fold-wiring.md:34`

**Issue:** Card 2 replaces `persistTransient`'s `appended` test with "this write recorded a verdict": history longer, or equal non-empty length with the last entry's `Repeats` differing from the re-read file.
I checked every no-verdict write that reaches `persistTransient` in `internal/shedengine/run.go`: the step-3 pause write (line 161), the step-3b resume write (line 178), and the cancellation arm (line 245) all pass `st.History` unchanged, and the producer-error arm with an empty outcome passes a copy `appendHistory` left unchanged.
Under the run lock Shed is the only history writer, so the re-read file's last `Repeats` equals `st.History`'s on each of those writes, while every fold increments it.
The discriminator separates correctly.
The test list, though, only covers the recompose side: the sentinel case asserts a folding step overwrites `activity.last`.
Nothing asserts that a pause or resume write landing right after a fold still carries `activity.last` forward.
That is the regression a wrong discriminator would cause, for example one keyed on `LastAt`, or one that treats any equal-length non-empty write as a fold.
D4 also accepts that a fold spans a pause/resume, and no test exercises that either.

**Fix:** Add a `step_fold_test.go` case: two exempt Stucks (one fold), then set `pause_requested` through `state.UpdateJSON`, and step to `paused`.
Assert history is unchanged and `activity.last` is still `"Wait → bounced to Wait"`.
Then step again: the step-3b resume write carries it forward, and the exempt Stuck folds into the same entry, so history stays at one entry with `Repeats` 2.

### [LOW] The Awaiting-interleave case folds nothing first and leaves the third step implicit

**Location:** `_lyx/plan/02-step-fold-wiring.md:40`

**Issue:** The acceptance criterion reads "Fold, then a Done or Awaiting, then another exempt Stuck".
Card 2's step-level case scripts exempt Stuck, Awaiting, exempt Stuck, so no fold precedes the Awaiting.
Card 1's unit table does cover the criterion.
The step-level case is still drivable: an Awaiting verdict persists state `awaiting` with `current_producer` unchanged, `stepLocked` does not short-circuit on `awaiting` (run.go:142), and the next `Step` goes through the step-3b resume write and re-calls the same producer.
The card says none of that, so an implementer could reach for `Run`, or conclude the case needs a manual state reset.

**Fix:** Script exempt Stuck, exempt Stuck, Awaiting, exempt Stuck, and assert three persisted entries: the first with `Repeats` 1, then the Awaiting, then a fresh exempt Stuck with no `Repeats`/`LastAt`.
State that the step after the Awaiting halt is a plain `Step` call, which re-calls `Wait` through the step-3b resume write.

### [LOW] Card 2's ImpactSummary omits history-length readers

**Location:** `_lyx/plan/02-step-fold-wiring.md:47`

**Issue:** "counted Stucks, routing and budget decisions are unchanged" is true.
It leaves out the one observable effect a fold has: it rewrites the file without lengthening history.
The readers of history length are `internal/shedverbs`' `history_length` (step, run and status envelopes), which `ly-drive` keys its advance check and repair cap on (D10), and `internal/battencli`'s `recentHistory` window.
`internal/loomengine/anomaly.go`'s `HistoryLength` and `countDoneEntries` read loom runs, and `internal/battenshed/innerrun.go:260` is the only `BudgetExempt: true` producer, so a loom history never folds.

**Fix:** Rewrite the ImpactSummary to name these readers: batten's `history_length` holds still across a folded wait, so ly-drive hands back an interrupted `Run-Shed` step (D10), and battencli's 20-entry window spans more of the wait.
loomengine's crash-resume and halt-title readers see no change, since loom producers never emit an exempt Stuck.

### [LOW] Stale battencli doc comment left behind

**Location:** `_lyx/plan/02-step-fold-wiring.md:3`

**Issue:** `internal/battencli/arm.go:568`, the `maxStatusHistoryEntries` doc comment, says "Run-Shed's self-bounce appends one per poll while the child runs, awaits approval, or waits for its driver, so the history grows past the 1440-bounce running budget".
After the fold that sentence is false for the awaiting and driver-exit waits.
It is the same claim the plan corrects in `docs/overview.md`.
The decision record keeps battencli's truncation behavior out of scope, not its comments, and its Scope names the doc comments that describe history appends.

**Fix:** Add `internal/battencli#maxStatusHistoryEntries` to card 2's `Edit:` targets, and a sentence to its Intent: reword the comment so only the running watch appends one entry per poll, bounded by its 1440-bounce budget, while budget-exempt waits fold. The constant's value and `recentHistory` stay unchanged.

### [NIT] The D9 case in card 4 depends on an unstated equal length

**Location:** `_lyx/plan/04-awaiting-resume-pending.md:40`

**Issue:** "its third Call with a new approval still resumes" proves D9's "regardless of the recorded length" only because the child's history length on that Call equals the recorded one.
At that length a matching identity would take the not-yet-re-stepped arm.
`awaitingStatus()` repeats its one status, so the test gets this for free, but the card never says so, and a later edit to the scripted statuses could quietly weaken the case.

**Fix:** Say that the third Call's child history length equals the length the marker recorded, so the new identity resumes despite the length that would otherwise mean "not re-stepped".

## Focus verdicts

- The persistTransient discriminator is sound for every no-verdict write (see F1 for the analysis). Only the test coverage is missing.
- The Awaiting case can be driven through `Step` (F2).
- The ImpactSummary states a real conclusion but misses the history-length readers (F3).
- Cards 3 and 4 against D7–D9: the not-yet-re-stepped reason names both D7 remedies, never names `lyx loom approve`, and never calls `lyx loom start` a re-delivery.
  Card 4's tests cover each D7–D9 acceptance criterion: the marker content, a longer child history, the old-layout marker, a new identity (F5 aside), not-parked, and many polls through `newRunShed`.
  Reading an unparseable second line as the old layout is not scope beyond D8.
  The parser has to answer that case somehow, and the old-layout answer (the re-approve hint, no spawn) is the conservative one and consistent with D8's rationale.
  A newline-free marker never matches and resumes, which is exactly how a torn identity-only write behaves today.
- Decision-record coverage: D1–D6 are carried by cards 1 and 2, D7–D9 by cards 3 and 4, and D10 by card 2's overview text, with ly-drive left unchanged.
  Each Constraint has a card: the single persist with the read slice unmutated (card 1's fresh-slice rule and input-unchanged test, card 2's one-`CommitStatus` test), the marker path through `approvalActedFile` (card 4), the fields on `HistoryEntry` for `ReadJSONStrict` (card 1), and docs in the same commit (card 2).
  The Notes' `HistoryEntry` and `persist` doc-comment updates are in cards 1 and 2.
