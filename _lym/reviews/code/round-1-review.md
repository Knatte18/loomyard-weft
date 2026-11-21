---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomshed/redfirstgate.go#redFirstHoldSendFailed
    summary: The new constant spells the record title "No test seam" as a literal instead of referencing gateslot.NoTestSeamTitle, against PATTERN-code-conventions' named-constant rule.
    fix: "In internal/loomshed/redfirstgate.go, build redFirstHoldSendFailed by concatenation with gateslot.NoTestSeamTitle, e.g. `... under a title opening \"` + gateslot.NoTestSeamTitle + `\"`, keeping the surrounding wording and quotes, so the title is declared once."
    origin: assist:assist
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomshed/redfirstgate.go#arrival
    summary: The hold line is returned only on the arrival that opens the hold, so when the wait loop fails to send it the writer is never told and the run can hold silently until its deadline.
    fix: "In redFirstGate.arrival, keep returning redFirstHoldLine(res) as Send on every pending answer of a hold until the writer has been told: track delivery apart from heldTree (for example a `told` flag set on the first arrival that finds the writer has moved off the boundary, or set when a later arrival sees a new turn), and keep the same-tree arrivals running no command. Add a step to TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands, or a shuttleengine-level test, where the first send fails and a later re-check sends the line, spending no attempt and running the check once."
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/loomcli/darnwiring.go#latestDarnOutcome
    summary: Now that a pass keeps the file, a later Darn-Write Stuck that wrote no findings (scope refusal, stop report, session Stuck, record check, terminal gate reason) names an earlier halt's stale findings and the prior-work note tells the writer to fix them.
    fix: "Name the kept file only when the latest Stuck's own gate wrote it. One way: in latestDarnOutcome, name findingsPath only when the file's modification time is not before the latest Darn-Write history entry's start (or the entry before it); another is to record the findings path on the Stuck's history entry and read it from there. Add a TestLatestDarnOutcome row: a Stuck, a Done, then a Stuck with the file kept and older than the latest Stuck's spawn, wanting no FindingsPath."
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/loomshed/redfirstgate.go#final
    summary: At a deadline or liveness finalize while held on an unchanged HEAD tree, Final runs the full red-first check again on the tree whose refusal the hold already judged.
    fix: "Keep the held refusal's verifytree.Result on redFirstGate beside heldTree; in final, after the exempt check, when heldTree is non-empty and verifytree.HeadTree(worktreeRoot) equals it, return redFirstFailure(held result) without calling verify. Pin it with a row that holds, then calls Final on the same tree and asserts one verify log."
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: consistency
    location: internal/shedrun/paths.go#GateFindingsFile
    summary: Four comments still describe the removed clearing or the old naming rule of the kept findings file.
    fix: "Update each to the new contract. internal/shedrun/paths.go GateFindingsFile doc: replace 'and the row's next call whose gates pass removes it' with 'and a later call whose gates pass leaves it in place'. internal/shedadapters/burlerreview.go gateFindingsPath field comment: replace 'the file cleared once the review's gate has passed' with 'the file an exhausted gate's findings are kept in'. internal/loomshed/doc.go (the prior-work note paragraph): say the note names the kept findings file only when the latest Darn-Write entry is a Stuck. internal/loomshed/darnwrite.go DarnOutcome.FindingsPath: say it is empty when no file is kept or the latest entry is not a Stuck."
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: design
    location: internal/loomshed/redfirstgate_test.go#TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands
    summary: Three red-first behaviours the diff adds have no failing row, namely the hold re-arming with a fresh line after a non-hold verdict, Final passing an exempt entry without running the check, and a timed-out run with exit 4 or 5 failing terminally instead of holding.
    fix: "In internal/loomshed/redfirstgate_test.go add: a hold-test step where a commit makes the check pass (or a green exit), then a commit brings exit 5 back, wanting pending with a non-empty Send; a TestRedFirstGate_VerdictPerLabelRecordAndExitCode row with final true on an entry without the bug label (and one with the No test seam record), wanting passed and no verify log; a row whose command exits 4 after exceeding a short injected timeout, wanting terminal failure, if verifytree's timeout is injectable there, otherwise a holdsForCommitOrRecord table over Result{TimedOut: true, ExitCode: RedFirstNoTestExit} and RunnerCrash."
    origin: lens:code-tests
  - id: F7
    severity: LOW
    class: design
    location: internal/loomcli/wiring_darn_test.go#TestPreflightTableError_KeepsANewerStencilRefusalsOwnWayForward
    summary: Only the helper is pinned; reverting PreflightStencils in wireDarn to the old inline config-clause wrap passes every test, though that call site is what issue 175 fixes.
    fix: "Replace the helper test with, or add, a wiring test that drives c.env.PreflightStencils over a darn table build that fails with a wrapped *stencilstore.NewerBoardCopyError (a newer-build stamp on one darn stencil in the test's stencils dir) and asserts the error names one 'way forward:' and no config-or-model-spec clause; keep one row for a non-stencil build error gaining the clause."
    origin: lens:code-tests
---

### [BLOCKING] The hold's send-failed way forward repeats the record title literal

**Location:** internal/loomshed/redfirstgate.go#redFirstHoldSendFailed

**Issue:** `redFirstHoldSendFailed` ends with `under a title opening \"No test seam\"`, spelling the title the gate matches.
`gateslot.NoTestSeamTitle` (`internal/gateslot/redfirst.go:24`) declares it, and `noSeamAsk` and `exempt` already reference it.
PATTERN-code-conventions' Named constants rule: a name that can go stale is one named constant, declared once and referenced everywhere, never a repeated literal.
A rename of the title would leave this way forward naming a title the gate no longer honours.
The round's constraints make every violation of a listed constraint BLOCKING.

### [MEDIUM] A failed hold-line send is never retried

**Location:** internal/loomshed/redfirstgate.go#arrival

**Issue:** `arrival` sets `heldTree` and returns `Send` only when `heldTree` was empty.
When `handleGatedBoundary` (`internal/shuttleengine/wait.go`, the pending branch) fails to send, it logs, keeps the entry pending and leaves the writer at the boundary.
Every later re-check on the same tree answers pending with an empty `Send`, so the loop has nothing to resend, and a new refusing commit sends nothing either.
The wait loop's contract expects a pending closure to keep offering its line until delivered; `redFirstHoldSendFailed` only reaches a log.
The writer, idle at the boundary, never learns to ask its parent for the record, so the hold the decision asks for can last until the run deadline.

### [MEDIUM] A later non-gate Stuck revives findings a pass already superseded

**Location:** internal/loomcli/darnwiring.go#latestDarnOutcome

**Issue:** Before the diff, a pass removed the file, so a Stuck after a pass could only find a file its own gate wrote.
Now the file survives the pass, and `latestDarnOutcome` names it for any latest Stuck.
Darn-Write ends Stuck without writing findings on a scope refusal, a stop report, a session Stuck and an invalid decision record (`internal/loomshed/darnwrite.go:144-199`), and `gateFailedReason` writes no file for a terminal gate reason.
After halt, resume, pass and a later such Stuck, `renderPriorWork` (`darnwrite.go:293-294`) tells the next writer to read and fix findings that a pass already cleared.

### [LOW] Final re-runs the check on the held tree

**Location:** internal/loomshed/redfirstgate.go#final

**Issue:** `verifytree.Verify` records passes only, so `final` runs `lym gate red-first` again at a deadline or liveness finalize even when HEAD's tree is the one the hold judged.
The verdict is the same; the cost is one more merge-base test run at finalize, the "check twice on one tree" the focus asks about.

### [LOW] Stale comments on the kept findings file

**Location:** internal/shedrun/paths.go#GateFindingsFile

**Issue:** `GateFindingsFile`'s doc says the row's next passing call removes the file.
The `gateFindingsPath` field comment in `internal/shedadapters/burlerreview.go` calls it the file cleared once the review's gate passed.
`internal/loomshed/doc.go`'s prior-work paragraph says the note names the kept file when one exists, and `DarnOutcome.FindingsPath` says it is empty only when none is kept.
After the diff no producer removes the file and `latestDarnOutcome` names it only for a latest Stuck.

### [LOW] Uncovered red-first hold transitions

**Location:** internal/loomshed/redfirstgate_test.go#TestRedFirstGate_HoldRunsNothingUntilACommitOrTheRecordLands

**Issue:** The hold test's steps stay held or end at the record, so nothing pins that `heldTree` resets and a later refusal sends the line again.
Every `final: true` row is a bug entry with exit 4 or 5, so `final`'s exempt branch is unreached.
The only timeout row exits 1, so dropping the `TimedOut || RunnerCrash` guard in `holdsForCommitOrRecord` would turn a timed-out exit-4 run from terminal into a silent hold with no failing test.

### [LOW] The Preflight wiring that issue 175 fixes is untested

**Location:** internal/loomcli/wiring_darn_test.go#TestPreflightTableError_KeepsANewerStencilRefusalsOwnWayForward

**Issue:** The new test calls `preflightTableError` directly.
`TestWire_DarnRecipe` reaches `PreflightStencils` but asserts no newer-stencil refusal, so restoring the old inline wrap at `internal/loomcli/darnwiring.go` in `wireDarn` passes every test.

## Rejected

- Assistant F4, a missing `Loosened-Check:` trailer for `TestClearGateFindings` and the changed findings-file rows: the decision asks to stop removing the file, the deleted test pins a function the diff deletes, and each changed row asserts the opposite contract by exact content, so no check is widened against behaviour the code still has.
- Tests lens 4, no test of a `HeadTree` read error while held: the branch only wraps and returns the git error, the same shape as every other read error in the gate.
- Tests lens 5, no test that plain entries keep `MayHold` false: `MayHold` only lifts the refusal of a pending result or a Final closure, which no plain closure returns, so a mutant setting it on every entry changes no observable behaviour.
- Handler suspicion that the orphan scratch sweep still deletes the halt's file: the sweep runs over the prime's scratch, needs a missing records directory and 24 hours of age, and a halted run keeps its records, so the commit's reproduced cause, `clearGateFindings` on a later pass, stands.

## Focus results

- Hold path: an arrival on the held tree runs nothing and spends nothing; a new commit runs the check once; the `No test seam` record lifts the hold because `exempt` is read first; Final turns the refusal into a failure with findings. Open issues are F2 and F4.
- `MayHold: final != nil` gives only the red-first entry hold rights, which `evaluateGate` accepts; every other entry resolves with nil Final and MayHold false, as before.
- `preflightTableError` withholds the config clause only for a `*stencilstore.NewerBoardCopyError`, found through `errors.As`, so a wrapped one is caught and every other error keeps the clause.
- Issue 174: the diff removes the cause the commit reproduced; the sweep is no factor (see Rejected). The reader-side regression is F3.
- `gateattempts_test.go`: the 63 deleted lines test only the deleted `clearGateFindings`; the retained rows check what their new names state.
- Root-Cause trailers on the three fix commits each name the defect their diff removes; no base-test check is loosened.
- The Darn-Write stencil change rewrites only the no-test and unjudged clauses and keeps the green-test rule and the `No test seam` ask.
