---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-teardown-skip-readings.md:80
    summary: Card 10 plans no test that reaches awaitRunLockOrPass, so the paused refusal with a parent lock record and its hub-entry form are reached only through the error constructor
    fix: >-
      In card 10 add to Create `plan:internal/pairteardown#TestAwaitRunLockOrPass` -> `func TestAwaitRunLockOrPass(t *testing.T)`.
      Plan it in an integration-tagged file beside `TestProbeQuietSeesTheGoDriverLock`, over a real task worktree whose run lock the test holds with `mustHoldLock`, with `td.sleep` set to a no-op so the `runLockWait` bound is spent without waiting.
      Rows call `awaitRunLockOrPass` itself: a hub entry (`paused` and `hubEntry` true) with a parent lock record naming the slug and a holder with no waiting marker returns an error wrapping `ErrDriverBusy` that names `lyx tender resume` with no slug and that it lifts every live entry of the hub;
      the same with `hubEntry` false names `lyx tender resume <slug>`;
      a live waiting marker carrying the holder's own pid returns nil with a parent lock record present;
      `paused` true and no parent lock record returns nil;
      a lock free at once returns nil without sleeping.
      Keep the wording asserts in `TestRunLockHolderPasses` as they are, and state in the card that `TestRunLockHolderPasses` pins the pure rule and the message while `TestAwaitRunLockOrPass` pins the holder, marker and parent-lock reads and the passing of both `pausedIn` readings into the refusal.
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/12-reed-down-ends-owned.md:39
    summary: Card 12 wraps card 10's run-lock refusal with lyx reed down in place of the removal, but the refusal text is built inside card 10 and ends with the removal, so reed down tells the operator to retry a removal
    fix: >-
      In card 10 give `runLockRefusal` a `retry string` parameter, the step the operator runs again, placed after `goos` (`slug, lockPath, goos, retry string, paused, hubEntry, parentLocked bool`), and give the `awaitOrPass` substrate and `awaitRunLockOrPass` a trailing `retry string` argument.
      Every "the removal again" and "retrying the removal once the holder ends" clause of the refusal text becomes that retry step, and the doubled "then then" in the paused-with-parent-lock sentence becomes "then".
      `EndSession` passes "the removal".
      In card 12 `EndBeforeSessionDown` passes "`lyx reed down`" and the sentence on a refusal of the run-lock wait drops "returned wrapped with the way forward `lyx reed down` again in place of the removal" for "returned as it is, its retry step already `lyx reed down`"; the `*runproc.SurvivorsError` wrap stays.
      Add to `TestRunLockHolderPasses` that each refusing row asserts the retry text it was given, and to `TestEndBeforeSessionDown` that the fake `awaitOrPass` sees the retry "`lyx reed down`".
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/11-fabric-remove-reconcile.md:51
    summary: The TestRemoveVerdict clause not naming lyx fabric remove as the step cannot hold, since the same card ends every refusal with then lyx fabric remove slug again
    fix: >-
      In card 11's `TestRemoveVerdict` paragraph replace "and not naming `lyx fabric remove` as the step, though the entry's `WayForward` holds the gone text" with "each refusal containing the other branch's `WayForward` text and not the gone branch's text (the sentence ending 'removes the pair and clears this record'), though the entry's `WayForward` holds the gone text; the refusal's closing 'then `lyx fabric remove <slug>` again' is expected in every row".
      The positive assertion on the other branch's text is what fails an implementation that reads `EntryOutcome.WayForward`.
  - id: F4
    severity: LOW
    class: consistency
    location: internal/shuttleengine/claudeengine/doc.go:244
    summary: Card 17 adds the waiting_held reading to the corpus case format but leaves the package doc that lists the closed set of readings at done, held or waiting
    fix: >-
      In card 17 add `internal/shuttleengine/claudeengine/doc.go` to the Edit list, and in the Tests paragraph on the corpus replay say its replay-corpus paragraph (the sentence "and the reading each turn end must get: done, held, or waiting with its outstanding tasks") gains `waiting_held`, a waiting turn end the wait loop judges as a plain turn end with an output missing, held at once.
---

### [MEDIUM] No named test reaches awaitRunLockOrPass's refusals

**Location:** `_lyx/plan/10-teardown-skip-readings.md:80` (and `_lyx/plan/12-reed-down-ends-owned.md:46`, `:49`)
**Issue:** The focus asks whether an input reaches the refusal naming `lyx tender resume` with no slug when a hub entry covers the run, and whether a named row reaches it.
The input exists: a hub entry covers the slug, a parent lock record names it, and the run lock is held past `runLockWait` by a step whose pid no live waiting marker carries (a step merging under the parent lock whose own Finalize push is the process waiting at the gate).
The named tests do not reach it.
`TestRunLockHolderPasses` calls the pure rule `runLockHolderPasses` and the error constructor `runLockRefusal`, which the rubric says does not reach the refusal.
`TestTeardown_CallSequence` and `TestEndBeforeSessionDown` fake the `awaitOrPass` substrate, so they see only the `hubEntry` value handed to it.
Card 12's `TestEndPairProcesses` installs a live hub entry but holds no run lock, so `awaitRunLockOrPass` returns nil there and reaches no refusal.
Nothing exercises the holder read (`shedverbs.RunningStepPID` or the registered driver), the waiting-marker read, the parent-lock read or the passing of `hubEntry` into `runLockRefusal`.
Card 12 does carry the input: `EndBeforeSessionDown` hands `awaitOrPass` both of `pausedIn`'s readings.

### [MEDIUM] Card 12's retry step cannot replace text card 10 has already built

**Location:** `_lyx/plan/12-reed-down-ends-owned.md:39`
**Issue:** Card 10's `runLockRefusal` ends each paused, unpaused and unknown-holder refusal with "then the removal again" or "retrying the removal once that pid ends", and its signature carries no retry step.
Card 12 returns that refusal "wrapped with the way forward `lyx reed down` again in place of the removal", and a wrap can only append, so `lyx reed down` would print both retry steps and name a removal for a verb that removes nothing.
`TestEndBeforeSessionDown` fakes `awaitOrPass` and asserts only that the error names `lyx reed down`, which the wrap alone satisfies, so no test sees the contradiction.
Card 10 line 67 also reads "then then the removal again".

### [LOW] TestRemoveVerdict's negative clause is unsatisfiable as written

**Location:** `_lyx/plan/11-fabric-remove-reconcile.md:51`
**Issue:** The focus asks where the stopped refusal's way forward is read from and whether the rows with the gone branch first fail an implementation that reads `EntryOutcome.WayForward`.
The card settles the source: `report.Branches`, the first stopping branch that is neither a gone branch of the slug nor an excluded busy branch, because `classifyEntries` fills the entry with the first stopping branch of the slug in target order, which is the gone code branch.
Each of the three gone-first rows asserts "the other branch's way forward" and so fails an implementation that reads the entry's text: that text is the gone branch's `FailedPushWayForward` sentence, which does not hold the diverged branch's text.
The added clause "not naming `lyx fabric remove` as the step" does not discriminate and cannot be asserted as a substring absence, since card 11 ends every refusal with "then `lyx fabric remove <slug>` again" and the gone text itself names `lyx fabric remove` of the pair.
An implementer reading the clause literally writes an assertion every correct implementation fails.

### [LOW] claudeengine's package doc keeps the old set of corpus readings

**Location:** `internal/shuttleengine/claudeengine/doc.go:244`
**Issue:** The focus asks whether the plan's tests and Edit targets cover every test and case file that replays `master-fork-running`'s last turn end through the real `Wait`.
They do: that case is the only corpus case with a shell-only turn end (the other cases list forks or nothing), `corpus_test.go` is its only replay, and card 17 lists `checkCorpusParse`, `corpusReplay.checkRead` and the `case.yaml`.
The in-package shuttle tests that carry a shell on a waiting turn end are either on card 17's list or unaffected: `TestWait_DialogNotice` ends at its five-minute timeout before the shell bound, `TestWait_LogsSessionStateBesideItsClassification` runs its shell row ungated with the output present at start or the next turn end inside the bound, and `TestAttach_LegacyAskingAndHeldRuns` is covered by card 17's own statement.
The one gap is documentation: the package doc says each case names "done, held, or waiting", and the card adds a fourth reading, `waiting_held`, without editing that sentence (`CLAUDE.md`: docs land in the same commit).

## Rejected

- Card 17, the replay under the new rule: not a defect. The last turn end of `master-fork-running` reads held at once (gated, an output missing, one unawaited payload shell, no awaited prefix in the replay's `Spec`), `notifyHeld` runs on the same tick because the replay sets no `HoldDeferral`, and the run still ends `OutcomeTimeout`; `checkCorpusStates` is unaffected because the fold does not change.
- Card 2, a read descendant leading its own group: not a defect. `TestEnd` names a row for a descendant whose group no live process reports or reports only through an older start time, with no group signal at SIGTERM, and the same state arising between SIGTERM and SIGKILL with no SIGKILL group signal, beside the rows for a recorded leader and for a descendant reused between the phases.
- Card 10, `busyError` and the `ownedProcesses` substrate: not a defect. `busyError` is called only from `waitQuiet`; every `waitQuiet` test builds its `Teardown` through `fakeSubstrates.teardown`, which the card edits, except `TestProbeQuietSeesTheGoDriverLock`, which the card names and gives a no-record function; the tmux-tier teardown tests build through `New`, which wires the real read, and none reaches the busy bound.
- Card 11, the source of the stopped refusal's text: settled as `report.Branches`, never `EntryOutcome.WayForward`; the ordering between the slug's own stopping branch and a shared one follows `classifyEntries` as the card's own sentence states, so it is not raised.
- Fix proposals F1, F2 and F4 add or change targets (a created test glyph, an Edit file, a `runLockRefusal` signature); `lyx loom validate-plan --overlay` over all of them together reported no blocking finding beyond the `{}` baseline.
