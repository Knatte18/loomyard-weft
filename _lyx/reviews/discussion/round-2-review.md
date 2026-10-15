---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 (spans paragraph) and D6
    summary: D5 suppresses gate spans only for the pending entry's own re-runs, but shuttle re-runs every earlier entry's closure on each recheck while a later entry is held, so an entry before parent-review records a gate span every few seconds of the hold and its gate line absorbs the hold's re-validation time
    fix: In decision-record.md D5, replace the sentence that begins with The evaluations a pending entry re-runs while held by one covering every entry, worded as - While an entry is held pending, the evaluations shuttle re-runs record no gate span for any entry of the gate, the held span covering them, so no entry gains a span per recheck. In D6, add after the gate-time sentence - An entry listed before a held one therefore shows only the evaluations outside the hold. In the acceptance criterion on gated runs, append - an entry listed before a held entry reports no gate span for the re-evaluations run during the hold.
  - id: F2
    severity: NIT
    class: consistency
    location: _lyx/discussion/support-log.md Turn 3, goal-scope bullet on the batten stage cell
    summary: The support log's lens-review bullet says D4 uses the dotted 4.7 form, an example round 1 corrected in D4 to 6.5 Plan-Write, so the two files disagree on the form's example
    fix: In support-log.md's Turn 3 bullet that begins with goal-scope — child number in batten's stage cell ambiguous, replace the words D4 uses the dotted `4.7` form with D4 uses the dotted `6.5` form.
---

### [MEDIUM] Earlier entries' re-evaluations during a hold are recorded as gate spans

**Location:** `_lyx/discussion/decision-record.md` D5, D6

**Issue:** D5 settles the held entry itself: its re-runs while held record no gate span, so its own gate and held spans never overlap, and D6's per-entry sum counts nothing twice.
That rule does not reach the other entries.
`evaluateGate` in `internal/shuttleengine/wait.go` runs the entries in list order from the first on every evaluation, with no skip for an entry that passed before.
While an entry is pending, the wait loop re-evaluates once per `gateRecheckEvery`, five seconds, through `handleGatedBoundary`.
Each of those evaluations re-runs the closure of every entry listed before the pending one.
In the loom recipe's Discussion-Write row, `discussion` stands before `parent-review`, which `internal/shedrecipe/entries_gate.go` builds as a `MayHold` entry that stays pending until the parent rules.
Under D5's "a gate span for each closure evaluation", `discussion` gains one gate span per recheck for the whole hold: about 720 spans in a history entry for an hour's hold, rewritten through `internal/state` on every save.
Its `gate discussion` line then shows the hold's repeated re-validation as gate time, which tells the operator the discussion check was slow when the time went to waiting on the parent.
No two lines add up to the same time, since D6 sums per entry, but the earlier entry's figure is wrong and the record has no bound.

### [NIT] Support log keeps the retired 4.7 example

**Location:** `_lyx/discussion/support-log.md` Turn 3

**Issue:** The goal-scope bullet on the batten stage cell says D4 uses the dotted `4.7` form.
Round 1 corrected D4's example to `6.5 Plan-Write`, since Run-Shed is batten's sixth row and Plan-Write loom's fifth entry, so the support log now cites an example the decision record rejects.

## Focus verdicts

- D5 and D6, held time: held is its own kind, summed into the gate line, and the held entry's re-runs are not double-counted; the defect is the earlier entries' re-runs, filed as F1.
- D5 Carriers: the carrier set is defined by the rows whose producer runs a gated shuttle run, and the producers named are the subset that copies a count today. The Burler review row is carried explicitly and matches the code: `reviewTable` in `internal/burlerengine/review.go` gates the reviewer on `ReviewGateEntry`, a `PassOnCap` entry with `reviewGateAttempts` (3). The Bouncer runs no gated shuttle run, so the first Auto-mode assumption's "covered by the round totals" agrees with the carrier rule. Not a defect.
- D7, open visit: D7's third sentence makes the open visit the rework visit when the last history entry is a gate bounce routed to its row, which needs no history index, and the acceptance criterion names "the open visit included". Not a defect.
- D3 and D4: under D3, batten numbers Worktree-Create 1 through Worktree-Teardown 8, so Run-Shed is 6 and Land 7. Loom numbers Preflight 1, Loom-Preflight 2, Discussion-Write 3, Discussion-Review 4, Plan-Write 5 and Plan-Review 6. Every example (`[x] 3. Discussion-Write`, `7. Land`, `6.5 Plan-Write`, `6.6 Plan-Burler-Review`) matches the shipped recipes. A child row inside a review segment takes its segment's number, and PR-Gate and PR-Rework, in a segment with no Bouncer row, each take their own number. Not a defect.
