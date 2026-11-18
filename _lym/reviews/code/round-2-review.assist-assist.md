---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/shuttleengine/relaunch.go:65
    summary: Reed reclaims the old strand inside AddStrand before the replacement run record is saved, so the record-first ordering remains broken.
    fix: Add a replacement-add mode through the reed seam that preserves the old strand and allocates a distinct available name without reclaiming it. Use that mode in relaunchForReprompt, save the new GUID, and only then remove the old strand. On a failed save remove only the replacement. Extend the existing resume test to exercise the real add reclamation semantics and assert that the old strand remains until the replacement record is saved.
    origin: lens:code-correctness
    carries: F4
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/stepstop.go:135
    summary: The holder guard accepts an absent holder even when Claude skipped an unreadable or undecodable registry entry, allowing an uncertain live run to be marked stopped.
    fix: Make the holder lookup used by sessionMayBeHeld preserve uncertainty from individual registry-entry read and decode failures through the provider seam. Return an error or an explicit uncertain result for those entries and retain the running record under Held. Extend the holder and step-stop behavior tests with an unreadable or malformed entry so the uncertain run cannot become respawn-eligible.
    origin: lens:code-correctness
    carries: F5
  - id: F3
    severity: MEDIUM
    class: scope
    location: internal/shedverbs/loop.go:471
    summary: Re-step reconciliation searches interrupted step records that the dead-loop stop never writes, so failed cleanup of the dead loop's child is not retried.
    fix: In the dead-loop reporting path persist the recorded in-flight child's trace ID as an interrupted step record before retiring its PID record, using the existing interruptedRecord writer without overwriting a completed child envelope. Have re-step reconciliation retrieve that identity. Extend the existing loop recovery tests to perform a dead-loop stop whose first cleanup fails and then a re-step, asserting cleanup retries the same child before starting the next child.
    origin: lens:code-scope
    carries: F7
prior:
  - id: F1
    status: resolved
    answer: Both startup proofs now read beyond the resume offset; the never-ready row seeds old startup signals.
  - id: F2
    status: resolved
    answer: Skills remain in the spec and both engines suppress only the skill-load turn.
  - id: F3
    status: resolved
    answer: The resume copies the rewritten manifest after saving its replacement record.
  - id: F4
    status: open
    answer: The explicit removal moved, but the real Reed add still reclaims the old name and removes its strand before the run record is saved.
  - id: F5
    status: open
    answer: Individual unreadable or undecodable Claude registry entries still yield a false absence answer to the new guard.
  - id: F6
    status: resolved
    answer: A nonempty resume-check warning refuses before Prepare.
  - id: F7
    status: open
    answer: The alternative lookup works for a surviving loop's interrupted child but not a dead loop, whose reporting path writes no interrupted step record.
  - id: F8
    status: resolved
    answer: The table covers the listed branches; a cleared binding correctly expects no removal because the strand is not removable.
  - id: F9
    status: resolved
    answer: The recorded-child panic row checks the hook trace ID and appended cause.
  - id: F10
    status: resolved
    answer: A failed save restores the prior in-memory state before removing the replacement; the separate old-strand ordering defect remains F4.
  - id: F11
    status: resolved
    answer: The function, package documentation and summary explicitly document replacement of prompt.md.
  - id: F12
    status: resolved
    answer: Finished runs have their own report field, rendering and assertion.
  - id: F13
    status: resolved
    answer: The summary states the young-run age guard and its expiry.
  - id: F14
    status: resolved
    answer: The specFor subtest invokes the orphan hook and checks its step tie through the receiver seam.
  - id: F15
    status: resolved
    answer: The empty-note row checks the unchanged cause without a trailing separator.
---

### [BLOCKING] Replacement registration removes the old strand too early

**Location:** internal/shuttleengine/relaunch.go:65

**Issue:** The real add operation has a removal side effect that the fake does not model.
For a dead, visible, childless strand holding the requested role or explicit name,
`reclaimDeadNameLocked` removes it before `AddStrandUnless` persists the replacement.
The evidence is `internal/reedengine/strand.go:258`, `:287`, `:644` and `:658`.
Shuttle saves the new GUID only at `relaunch.go:79`.
A kill between those operations or a failed save leaves the running record naming the removed old GUID.
This still violates the record-first rule in `PATTERN-shuttle-provider-seam`.

### [MEDIUM] Unknown registry entries bypass the holder protection

**Location:** internal/shuttleengine/stepstop.go:135

**Issue:** Claude's `sessionHolder` skips individual read and decode failures and returns no holder.
The evidence is `internal/shuttleengine/claudeengine/resume.go:125` through its return at `:134`.
Unlike `checkResumable`, this lookup does not communicate uncertainty.
The new step-stop guard accepts that answer and records the run stopped.
Attach then treats the terminal record as respawn-eligible without its young-running-record guard.
The earlier F5 protection therefore remains incomplete for an unreadable live holder's registry entry.

### [MEDIUM] Dead-loop cleanup cannot be retried from the selected records

**Location:** internal/shedverbs/loop.go:471

**Issue:** A dead loop leaves its child trace in the PID record and an in-flight step record.
`loopExitStop` writes the failed status and short loop envelope but no interrupted step record
(`internal/shedverbs/loopwait.go:623` through `:681`).
Delivery then retires the PID record.
The new lookup accepts only records with `kind: interrupted` and a trace ID.
After interruption-time cleanup fails, the re-step finds nothing or an older unrelated interruption.
The test seeds the desired interrupted record directly, so it does not exercise the dead-loop writer.
The correctness and scope lenses independently identified this same remaining F7 gap.

## Validation

The tier-0 gate was attempted for the changed packages.
It could not acquire a slot because the sandbox exposes the hub's `_bolt/.lym/gate` directory read-only.
No tests ran in this review.
The findings are grounded in the source paths above.
The test lens's additional coverage observations are below the incremental round's new-finding threshold.
