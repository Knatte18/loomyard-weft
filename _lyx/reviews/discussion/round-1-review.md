---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md (D7, Auto-mode assumptions)
    summary: D7's not-picked-up reason names lyx loom start as the re-delivery remedy, but start refuses a live driver whose park marker the resume already removed
  - id: F2
    severity: LOW
    location: _lyx/discussion/decision-record.md (D7, D9, Open risks Mixed binaries)
    summary: The new marker layout is never stated, and an older binary's whole-file compare against it fails and re-attempts a resume; Mixed binaries covers only the status file
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md (Open risks history_length) and support-log.md Q10
    summary: The history_length open risk defers a fact Discussion can settle; batten wires no interrupt policy, so an unchanged read hands back
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md (Open risks Activity recomposition, Notes persistTransient line)
    summary: The Activity recomposition risk and the persistTransient note restate D6, and the stale-value hazard they describe is unreachable
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md (Goal, D1 rationale)
    summary: The Goal reads awaiting-only while D2 folds every exempt Stuck, including the bounded driver-exit grace wait, and D1 calls every exempt Stuck unbounded
  - id: F6
    severity: NIT
    location: _lyx/discussion/decision-record.md (Notes for the plan writer)
    summary: Notes carry Plan-time code detail and a docs quote that duplicates Scope
  - id: F7
    severity: NIT
    location: _lyx/discussion/decision-record.md (D7)
    summary: D7's no-masking rationale rests on the child being a loom run rather than on D1 never folding an Awaiting entry, and it names only the equal and greater length cases
---

### [MEDIUM] D7's remedy text does not work for the live-driver case

**Location:** `decision-record.md` D7 first sub-bullet; Auto-mode assumptions "A resume the child never picks up …"; `support-log.md` Q9.

**Issue:** D7 has the not-picked-up reason say that `lyx loom start` in the task worktree re-delivers the resume.
`internal/loomcli/driverresume.go`'s `resumeParkedDriver` removes the park marker once the resume line is sent.
After that, `lyx loom start` (`internal/loomcli/start.go`, `runDriverSpawnAndWait`) finds a live strand, no marker, and a run halted at a hand-back, so it refuses with `driver_not_parked` and re-delivers nothing.
Start helps only when the driver strand has ended: then it spawns a fresh driver, which re-steps the run.
The usual reason a resume is not picked up is a live driver that has not acted on the typed line, and for that case the reason text names a remedy that fails.

**Fix:** The reason says the resume was delivered and the child's driver has not re-stepped yet.
If the child's driver has ended, `lyx loom start` in the task worktree starts a fresh one.
If the driver is still live, the operator inspects its pane.
Apply the same wording to the Auto-mode assumption and Q9.

### [LOW] Marker layout unstated; the reverse mixed-binary direction is unrecorded

**Location:** `decision-record.md` D7, D9, Open risks "Mixed binaries".

**Issue:** `callAwaiting` today compares the whole marker file byte-for-byte with `approvalIdentity`'s line (`string(acted) == identity`).
Any layout that adds the length inside that file fails an older binary's compare.
The older binary then calls `Spawn` again.
While the resume is still pending, the child's bootstrap refuses as not parked, so the older binary reports "not acted on yet" on every poll instead of double-resuming.
Once the child parks at its next hand-off, the older binary resumes it once on the already-acted approval, then rewrites an identity-only marker.
In practice the outer status file almost always carries `repeats` by then, so the older binary fails strict decode before reaching the marker.
D8 covers the other direction, but the record never states the layout the plan writer must build, and Mixed binaries names only the status file.

**Fix:** D7 states the layout: the identity line unchanged, followed by a second line holding the decimal length.
Mixed binaries records the reverse marker direction and its bounded consequence as accepted.

### [LOW] The history_length open risk is closable in Discussion

**Location:** `decision-record.md` Open risks, first entry; `support-log.md` Q10.

**Issue:** The entry asks the plan writer to confirm that batten's `interrupt_policy` makes an unchanged read safe.
`internal/battencli/arm.go`'s `specFor` wires no `InterruptPolicyFor` hook; only `loomcli` does.
Batten's `interrupt_policy` is therefore always `""`, and `ly-drive`'s Interrupted-invocations rule hands back unconditionally on an unchanged read.
An interrupt that lands after a fold's persist now hands back where it used to continue, which is the conservative direction.
That is a settled fact, not a risk, and the constraint the plan writer must honour (the fold does not advance `history_length`, and `ly-drive` is untouched) appears nowhere in the decision record's Decisions or Constraints.

**Fix:** Replace the open risk with a decision that `history_length` stays frozen during a folded wait, stating both consequences and accepting them.
Mark Q10 resolved.

### [LOW] Activity recomposition risk restates D6 and overstates its hazard

**Location:** `decision-record.md` Open risks "Activity recomposition"; Notes "`persistTransient`'s `appended := …` is the D6 hazard".

**Issue:** Both lines restate D6's requirement.
The Open risk also claims a missed D6 leaves `activity.last` "stale but plausible-looking".
A fold targets an exempt Stuck entry by the same producer, so that entry was persisted with the same `routedTo` (`def.OnStuck`), and `composeActivity` produced the same "`<producer>` → bounced to `<routedTo>`" string.
Every non-appending write since then (pause, resume, the outcome-less failure write) carries `prevLast` forward unchanged, and nothing outside `persistTransient` writes `Activity`.
The carried value therefore equals the recomposed value on every reachable path.
D6 remains a sound explicit rule, since a fold is a history change, but the risk it guards is not live.

**Fix:** Delete the Open risk entry and the Notes line.
D6 and its acceptance criterion stay, which satisfies the completeness-before-leanness test.

### [LOW] Goal and D1 rationale understate the fold's reach

**Location:** `decision-record.md` Goal; D1 rationale.

**Issue:** D2 folds every budget-exempt Stuck, and `battenshed` emits one for the awaiting arms and for the done arm's driver-exit grace wait.
The docs criterion names the driver-exit poll, but the Goal describes only the human hand-off wait.
D1's rationale says exempt Stucks are "the only unbounded source" of growth, yet the grace wait is bounded by `driverExitGrace`; only the awaiting wait is unbounded.

**Fix:** The Goal says the fold covers every budget-exempt wait, the driver-exit grace wait included.
D1's rationale says exempt Stucks are unbudgeted waits, and the awaiting one is unbounded.

### [NIT] Notes carry Plan-time detail and a duplicate docs quote

**Location:** `decision-record.md` Notes for the plan writer.

**Issue:** The writer stencil leaves code-level facts to Plan time.
The `nowRFC3339` line is that class of fact.
The `docs/overview.md` line repeats the Scope Docs bullet's own quote.

**Fix:** Drop both lines; the remaining pointers stay as a head start.

### [NIT] D7 rationale and length cases

**Location:** `decision-record.md` D7.

**Issue:** The rationale says folding never masks the growth because the child is a loom run with no exempt Stucks.
That holds, but it is contingent.
The structural reason is that a re-await appends an Awaiting entry, which D1 never folds, whatever the child emits.
D7 also names only the equal and greater length cases.

**Fix:** Ground the rationale on D1's rule, and make the second case "any other length".
