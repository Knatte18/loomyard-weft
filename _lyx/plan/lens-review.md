# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Merged duplicates are listed once, naming every lens that raised them.

- coverage BLOCKING, way-forward BLOCKING, guard-reach MEDIUM 1 — the capture start returns on the lock alone, so the child's `Check` can read a status the start already moved on and capture nothing: fixed as proposed — card 5's `StartCapture` returns only once the capture record reads `running` with its own child's pid, written after `Check` passed; cards 7 and 9 say so; `TestStartCapture` gains the rows.
- guard-reach MEDIUM 2 — `lyx drift capture` mistakes another capture's lock for its child's: fixed as proposed — `StartCapture` refuses while the lock is held before spawning (card 5), and the pid-matched record wait (above) confirms its own child; row added.
- forced-edits BLOCKING 1 — `loomshed.InterruptPolicies` lacks the Drift rows, which `interruptpolicy_meta_test.go` requires: fixed as proposed — card 14 edits `internal/loomshed#InterruptPolicies`, all three reinvoke.
- forced-edits MEDIUM 2, row-validity BLOCKING 1 — `wantSequenceOrder` is derived, so the skipped entries need `deriveSequenceOrder`: fixed as proposed — card 14 targets `internal/loomrecipe#deriveSequenceOrder` with a skipped branch for the Drift rows.
- forced-edits MEDIUM 3 — the overview's execution stack, which the record's Docs scope names, is untouched: fixed as proposed — card 14 adds the execution-stack bullet.
- forced-edits MEDIUM 4 — `internal/loomrecipe/doc.go` tails go stale: fixed as proposed — card 14 edits it.
- forced-edits LOW 5 — the timing report's phase tables miss the Drift rows: fixed as proposed — card 14 edits `internal/reportengine#loomPhases` and `#darnPhases`.
- forced-edits LOW 6, row-validity LOW 7 — `fixedGateLists`' comment pins "eight": fixed as proposed — card 3 drops the count.
- forced-edits LOW 7 — `shedadapters/doc.go` keeps the narrow prefix description: fixed as proposed — card 11 extends it.
- forced-edits LOW 8 — card 13's test-seam glyphs unconfirmed: disputed, premise false — `lyx loom validate-plan` resolves `internal/pairteardown#fakeSubstrates.teardown` and `#TestTeardown_CallSequence`, and the recording fake is the card's own change, now with a scriptable error.
- coverage MEDIUM — the flag turned off at an entered row skips neither row, argued without a row: fixed as proposed — `TestDriftRows_EntryGate` gains a row with `EnteredRow` set and the flag false, asserting the row is called, not skipped.
- coverage MEDIUM — a row-started capture outlives its step on Windows and teardown ends it: fixed differently — the proposed Windows row cannot run under any gate of this hub (Linux only, Shared Decision "Windows job behavior proved outside this hub's gates"; decision record Scope Out names `csharp-sandbox-hub` as the end-to-end proof), so instead `TestCapture_OutlivesCaller` gains a Linux row in which `runproc.End` over the pair's slug ends a capture and its command.
- coverage MEDIUM — no row asserts the decision prefix on the dirty, verify and undeclared-compare Stucks under `full`: fixed as proposed — `TestDriftVerify` gains those rows.
- coverage MEDIUM — undeclared `capture`/`compare` refusals unreached at the row and verbs, and the side capture's step unstated: fixed as proposed — card 11 row for step 5, card 7 names `Config.Step("capture")` on `--side` and a tier1 row for `lyx drift compare` with no compare declared.
- coverage MEDIUM — `CheckVerifyRow` reaches no test: fixed as proposed — `TestAcceptWithdraw` carries its rows.
- coverage MEDIUM — the resume detection's `LastAt` and negative branches have no row: fixed as proposed — `TestDriftVerify` gains both.
- coverage MEDIUM — a held lock without `WaitCapture` (loom) has no row: fixed as proposed — `TestDriftBaseline` row added.
- coverage LOW — the capture's skip file is never asserted: fixed as proposed — `TestRunCapture` row added.
- coverage LOW — the failed foreground fallback's Stuck reason unasserted: fixed as proposed — `TestDriftBaseline` row asserts it.
- coverage LOW — the commit pathspec unchecked: fixed differently — the proposal implied a wiring test of `driftDeps`; `Commit` is `CommitAnchoredPaths` over `DirRel()` alone, so `TestLoadConfig` pins `DirRel` as `_lyx/drift` with every durable path under it, which is what the pathspec covers.
- coverage LOW — the local layer refusing the executable keys untested: fixed as proposed — `TestLoadConfig` row added.
- coverage LOW — small branches untested: fixed as proposed — the not-initialized mapping (card 1), `ProbeLock` on a missing directory (card 5), the probe-error return (card 12), `lyx drift status` (card 7) and the baseline table's two subjects (card 8) each gain a row.
- coverage LOW — the read-after-probe order unpinned: fixed as proposed — `TestDriftAwait` row added.
- row-validity MEDIUM 2 — the capture-kill error leaves the loop lock held, and the call strings change: fixed differently — the proposal joins `releaseLoopLock(false)`; card 13 instead moves the capture kill before the loop seize, right after the owned-process end, so its error has no lock to release, and names the call-string updates and a scriptable error.
- row-validity MEDIUM 3 — the breakaway rows are unreachable on Linux: fixed as proposed — `SpawnRequest` gains a `breakawayRefused` seam, nil meaning `proc.IsBreakawayRefused`.
- row-validity MEDIUM 4 — `TestCapture_OutlivesCaller` lacks the seed, the origin record and a way to fail once: fixed as proposed — the card names the seed, the origin record's parent branch and a marker file the command fails without.
- row-validity MEDIUM 5 — the gate-loop harness is an unlisted helper: fixed as proposed — card 11 declares `gateLoopSeats` and states the `GateOutcome` it builds and how the producer reads it.
- row-validity LOW 6 — `TestScopeRecord_Unruled` is a per-symbol test: fixed as proposed — dropped; `TestDriftVerify`'s full-scope rows reach `Unruled`.
- way-forward MEDIUM — a resume from a dirty-tree or test-gate halt compares past what it guarded, and the halts name no way forward: fixed as proposed — card 11's new step 6 re-checks the tree and re-runs the verify gate when HEAD moved past the attempt head, before any compare; the exhaustion Stucks name their way forward; rows added.
- way-forward MEDIUM — the dirty capture halt names committing, which moves HEAD off the branch point: fixed as proposed — cards 5 and 9 name cleaning the paths, recorded as Shared Decision "a dirty capture halt names cleaning" against D3's wording under PATTERN-refusal-way-forward.
- way-forward LOW — a child that exits first is named by an empty log directory: fixed as proposed — the child writes its own stdout and stderr to `capture-child-<stamp>.log`, which the error names.
- way-forward LOW — a `StartCapture` failure's return is unstated: fixed as proposed — card 9 returns it as an error, a resumable stop, with a row.
- way-forward LOW — the run-budget refusal tells a caller at a halt the agent's way forward: fixed as proposed — `TakeRun` returns `ErrRunsSpent` and card 7's verbs word it per caller.
- guard-reach LOW 3 — the Windows capture kill skips a gone worktree: fixed as proposed — card 13 computes the job name from the pair's path and calls it for a gone worktree too.
