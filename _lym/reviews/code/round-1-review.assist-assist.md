---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: internal/loomshed/redfirstgate.go:208
    summary: The new recovery message duplicates the decision-title constant.
    fix: Build redFirstHoldSendFailed by concatenating gateslot.NoTestSeamTitle instead of spelling No test seam again; preserve the surrounding quotes and wording.
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/loomshed/redfirstgate.go:111
    summary: A failed first hold notification is consumed before delivery and never retried.
    fix: Retain an undelivered pending notice in shuttleengine.handleGatedBoundary until sendWithin succeeds, retrying it while the entry remains pending and discarding it when that entry settles. Keep red-first verification cached on the held tree. Add a regression with the red-first hold, a failed first send and a successful retry that spends no attempt and runs no second verification.
    origin: lens:code-correctness
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/loomcli/darnwiring.go:145
    summary: A later non-gate Stuck outcome revives findings from a gate that already passed.
    fix: Track whether retained findings are still active for the latest gate outcome separately from their historical file. Clear that active association on a passing gate without deleting the file, and make latestDarnOutcome require the association before returning FindingsPath. Cover an initial gate failure followed by a passing gate whose Darn-Write wrapper returns Stuck for a stop marker or invalid decision record.
    origin: lens:code-correctness
  - id: F4
    severity: MEDIUM
    class: consistency
    location: internal/shedadapters/gateattempts_test.go:129
    summary: Removed base-test assertions have no required Loosened-Check trailer.
    fix: Add a Loosened-Check trailer to a new commit in the review range naming the deleted TestClearGateFindings checks and replaced pass-time deletion assertions in the producer and recipe tests. State the measured cause from the reproduced halt/pass sequence, including its observed deletion, and explain the replacement retention assertions. Keep the intended retention behavior.
    origin: handler
  - id: F5
    severity: LOW
    class: consistency
    location: internal/shedadapters/burlerreview.go:160
    summary: Two affected contracts still say a passing gate removes retained findings.
    fix: Update the gateFindingsPath field comment in burlerreview.go and the GateFindingsFile doc comment in internal/shedrun/paths.go to say a later passing call retains the historical file.
    origin: lens:code-affected-set
---

### [BLOCKING] Use the declared decision title in the recovery message

**Location:** internal/loomshed/redfirstgate.go:208

**Issue:** The new constant embeds `"No test seam"` independently of `gateslot.NoTestSeamTitle`.
The exemption parser and `noSeamAsk` use the owning constant in `internal/gateslot/redfirst.go:24`.
`PATTERN-code-conventions` requires a name that can become stale to have one declaration and references elsewhere.
The round requires every listed constraint violation to be BLOCKING, regardless of its size.

### [MEDIUM] Retry a hold notice that never reached the writer

**Location:** internal/loomshed/redfirstgate.go:111

**Issue:** The first refusal sets `heldTree` before shuttle attempts the carried `Send`.
On a transient send failure, `handleGatedBoundary` clears the memo and leaves the writer at the boundary.
The next same-tree arrival returns Pending without Send at `redfirstgate.go:93`.
A new tree that also refuses suppresses the notice too, because the earlier held tree remains nonempty.
The writer can wait until the deadline without receiving the instruction to contact its parent.
`TestGatePending_FailedSendWarnsAndStaysPending` supplies Send repeatedly, so it misses this integration failure.
This consolidates the correctness and tests lenses' reports of the same defect.

### [MEDIUM] Distinguish gate failures from other Darn-Write stops

**Location:** internal/loomcli/darnwiring.go:145

**Issue:** A latest Stuck entry does not establish that its gate produced the retained findings.
After an earlier gate failure, a later session can pass its gate and then return Stuck for a stop marker.
The post-session decision-record check can also produce Stuck (`internal/loomshed/darnwrite.go:167–205`).
The passing producer now preserves the earlier file, and the new predicate attaches it to that unrelated halt.
`renderPriorWork` then tells the next writer to fix those resolved findings (`darnwrite.go:294`).
The former pass-time deletion prevented this particular sequence from resurfacing old findings.
This consolidates the correctness and affected-set lenses' reports.

### [MEDIUM] Record the measured cause of removed checks

**Location:** internal/shedadapters/gateattempts_test.go:129

**Issue:** The range deletes `TestClearGateFindings` and replaces existing assertions that passing gates remove files.
The replacement retention assertions fit the requested behavior, and the removed helper has no remaining caller.
Nevertheless, the rubric explicitly requires a `Loosened-Check:` trailer for an assertion removed from a base test.
No commit in `57015a52cd1f436ce96f5137889045e9a2ce5d7d..HEAD` carries that trailer.
The retention commit records the 20:48:28 halt and 20:49:51 passing call as its reproduction;
its `Root-Cause:` trailer does not satisfy the separate trailer requirement.

### [LOW] Correct the remaining deletion promises

**Location:** internal/shedadapters/burlerreview.go:160

**Issue:** The field comment still calls this the file cleared once the review's gate passes.
`internal/shedrun/paths.go:303–304` likewise promises removal on the row's next passing call.
Both describe the behavior this diff removes.

## Focus results

The changed code traces to the added Scope decision replacing the corrupted Goal body.
`lym board get halt-escalation-messages` returned the recorded invalid-JSON error.
The issue bodies and the added decision supplied the scope.

Ordinary same-tree hold arrivals do not rerun verification or spend a re-prompt attempt.
A changed tree reruns the check, and the parent's matching decision releases the hold.
Final deliberately evaluates the refusal as a failure with findings.
The MayHold wiring is accepted by shuttle; the other plain gates retain their earlier contract.
The hold's delivery failure is F2.

The typed Preflight check preserves a wrapped newer-stencil error's own way forward.
Other errors retain the generic clause through the unchanged fallback.
The stencil change retains the test requirement and the parent's decision procedure.

The retention commit identifies pass-time deletion as the reproduced cause of issue #174.
The named scratch sweep resolves the prime before cleaning, while the task file lives under its task anchor.
The removed deletion helper addresses that identified cause; the remaining reader regression is F3.
The three substantive fix commits' Root-Cause trailers name defects their diffs change.

The unmapped helpers are exercised by the red-first table and hold-transition tests.
No missing-test finding follows from the advisory reach-map entries alone.
The changed tests introduce no production-duration wait.

## Validation

Attempted `timeout 3600s lym gate test --tier 1 ./internal/loomshed` in the foreground.
It could not acquire a gate slot because the sandbox makes Bolt's gate and process-record directories read-only.
No tests ran, and this environmental failure is not reported as a code defect.
The review therefore rests on the supplied gate reports and source-path tracing, without a fresh test result.
