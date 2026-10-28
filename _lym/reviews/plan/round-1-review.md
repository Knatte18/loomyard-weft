---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/01-finalize-merges-in-place.md (Docs paragraph; internal/landingshed/doc.go:120)
    summary: The HandsBack paragraph of internal/landingshed/doc.go still lists a loom conflict and a large merge as reroute causes, and neither card 1's nor card 2's Docs line drops them
    fix: In card 1's Docs paragraph, after the sentence on internal/landingshed/doc.go's Finalize merge paragraph, add that the HandsBack paragraph's list of what Finalize routes back through the recipe's reroute edge drops a loom conflict and a large merge and keeps only a spent or over-cap fix. internal/landingshed/doc.go is already a card 1 target, so no target changes.
  - id: F2
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/01-finalize-merges-in-place.md (Intent; internal/landingshed/finalize.go Finalize.Call doc)
    summary: Card 1 rewrites only Finalize.Call's retry sentences, but the Call doc's merge sentence still says a loom Finalize aborts a conflict and a large merge reroutes with no gate run
    fix: In card 1's Intent, extend the sentence on Finalize.Call's doc to say its merge sentence is rewritten too. It should say Finalize merges the parent through its resolver, resolves a conflict with the conflict session under the lock, and gates the merged tree, with no reroute on a conflict or a large merge. Keep the existing retry wording. finalize.go is already a card 1 target, so no target changes.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/02-escalate-halt-prefix.md (Edit list; internal/battenshed/deps.go:293, internal/battenshed/innerrun_binaryrestep.go:53)
    summary: Card 2 widens batten's HandBack seam to the escalate halt but leaves the InnerRunDeps.HandBack field doc and the binaryRestep doc calling it a landing halt only the orch's resume lifts
    fix: Under card 2's Edit, add `internal/battenshed#InnerRunDeps` and `internal/battenshed#innerRunProducer.binaryRestep` after `internal/battenshed/doc.go`. Add a Docs sentence saying both comments name a landing halt or a loom or darn escalate halt as what the seam marks and the re-step leaves alone. The combined validate-plan overlay passes.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/01-finalize-merges-in-place.md (Edit list; internal/loomcli/wiring.go:955)
    summary: loomRerouteCapWayForward keeps a clause for a parent-branch reroute, a cause D1 removes, and two tests pin the clause; no card lists the constant or the tests
    fix: Under card 1's Edit, add `internal/loomcli#loomRerouteCapWayForward`, `internal/loomcli#TestLoomRerouteCapWayForward_IsRowNeutral` and `internal/loomcli#TestLoomRerouteCapWayForward` after `internal/loomcli/doc.go`. In card 1's Intent, say the constant drops its clause for a parent-branch reroute. In the Tests list, say both tests stop asserting "parent branch has settled" and keep their other fragments. The combined validate-plan overlay passes.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:way-forward
    location: _lym/plan/02-escalate-halt-prefix.md (Intent, the pawl way forward of fixerOutcome.haltReason)
    summary: The pawl escalate way forward adds a clause D3 does not carry, and that clause is false at a pawl Parent-Align whose Code-Review perch budget is above 0
    fix: "In card 2's Intent, end the pawl way forward at: settle the conflict on the task branch, then `lym loom resume`. That is the text D3 records. Delete the clause \"; a settle past the fixer's caps halts over-cap, and the resume after that halt accepts the commits\" and the sentence after it that explains the clause."
  - id: F6
    severity: LOW
    class: design
    origin: handler
    location: _lym/plan/03-arbiter-judges-escalate-first.md (Tests, TestGoDriver_Settle)
    summary: The planned arbiter_max_spawns row repeats a behaviour an existing TestGoDriver_Settle row already pins, so it is a redundant test row
    fix: "In card 3's Tests, delete the row sentence that begins \"One more row sends the escalate-prefixed stop under the same seam to an arbiter that answers `arbiter_max_spawns: 0 turns the arbiter off`\". The existing row \"an arbiter off refusal at an awaiting stop with a notice still relays it\" (internal/sheddriver/drive_test.go:301-304) pins that an arbiter error hands back. The new judges-first hand-on row pins that the escalate stop reaches the arbiter."
---

### [LOW] landingshed doc's HandsBack paragraph keeps the removed reroute causes

**Location:** `_lym/plan/01-finalize-merges-in-place.md`, Docs; `internal/landingshed/doc.go:120`

**Issue:** `internal/landingshed/doc.go:120` reads "a loom conflict, a large merge or a spent or over-cap fix at Finalize routes back through the recipe's reroute edge".
After D1 only a spent or over-cap fix does.
Card 1's Docs line covers only the Finalize merge paragraph (lines 41–46).
Card 2's HandsBack-paragraph edit is about the escalate.
So no card drops the two dead causes, and D5's "docs follow the behaviour" is only partly carried.

### [LOW] Finalize.Call's doc keeps the loom abort and large-merge reroute

**Location:** `_lym/plan/01-finalize-merges-in-place.md`, Intent; `internal/landingshed/finalize.go` `Finalize.Call` doc

**Issue:** The doc reads: "It then merges the parent branch into the task worktree and judges the merge: a conflict a loom Finalize aborts, a conflict a darn Finalize resolves, and a merge large by its conflict or by files both sides changed reroute, with no gate run."
Card 1's Intent assigns only the retry sentences of `Finalize.Call`'s doc to be rewritten, so a writer following the card can leave this sentence false.

### [LOW] Batten's HandBack field and binaryRestep docs still say landing halt only

**Location:** `_lym/plan/02-escalate-halt-prefix.md`, Edit list; `internal/battenshed/deps.go:293`, `internal/battenshed/innerrun_binaryrestep.go:53`

**Issue:** After card 2, `restepSkips` marks the escalate halt too, so these two comments become false:

- `InnerRunDeps.HandBack`: "reports whether the child's blocked status is a landing halt only the orch's resume lifts".
- `binaryRestep`: "is not a landing halt that deps.HandBack marks for the orch".

The escalate halt is not a landing halt, and the arbiter's resume can lift it.
Card 2 edits only `internal/battenshed/doc.go`.

### [LOW] The reroute-cap way forward keeps a dead parent-branch cause

**Location:** `_lym/plan/01-finalize-merges-in-place.md`, Edit list; `internal/loomcli/wiring.go:955`

**Issue:** `loomRerouteCapWayForward` opens its causes with "for a parent-branch reroute, once the parent branch has settled".
After D1, Finalize reroutes only through `keepFixerCommits`, for a spent or over-cap fixer, and Parent-Align has no reroute edge.
So no new reroute-cap halt can quote a parent-branch cause.
A halt an older build made has already persisted its own text, so the constant serves only new halts.
Two tests pin the clause: `TestLoomRerouteCapWayForward_IsRowNeutral` (`wiring_test.go:1052`) and `TestLoomRerouteCapWayForward`.

### [LOW] Pawl escalate way forward adds a clause D3 does not carry, and the clause is false at one row

**Location:** `_lym/plan/02-escalate-halt-prefix.md`, Intent (pawl way forward of `fixerOutcome.haltReason`)

**Issue:** D3 fixes the pawl way forward as "settle the conflict on the task branch, then `lym loom resume`".
The card appends "a settle past the fixer's caps halts over-cap, and the resume after that halt accepts the commits", together with a sentence that explains it.

The clause holds at Finalize: the escalate's record keeps `Baseline`, so a passing gate judges the change, and with `RerouteReviewed` nil the run halts over-cap.
It does not hold at a pawl Parent-Align whose Code-Review perch budget is above 0:

- the change goes to the Code-Perch unjudged (`internal/landingshed/doc.go:113`);
- `judgeFixerChange` judges only when no review follows (`parentalign.go:421–424`);
- pawl's Parent-Align `on_done` is Code-Bouncer (`pawl-recipe.yaml:57–59`).

The halt text would then promise a halt that never comes.

### [LOW] Card 3's arbiter-off row duplicates an existing driver test row

**Location:** `_lym/plan/03-arbiter-judges-escalate-first.md`, Tests

**Issue:** The planned row sends the escalate-prefixed stop to an arbiter whose `Judge` errors with `arbiter_max_spawns: 0 turns the arbiter off`, and expects a hand-back.
How the driver treats an arbiter error does not depend on the route that reached the arbiter.
The existing row "an arbiter off refusal at an awaiting stop with a notice still relays it" (`internal/sheddriver/drive_test.go:301–304`) already pins it.
The card's own new row, the judges-first escalate stop with an arbiter hand-on, already pins that the escalate stop reaches the arbiter with no tackler call.
So the extra row covers no new behaviour.

## Rejected

- guard-reach finding 1, `EscalateHalt`'s `retiredMergeInName` branch is unreachable: D3 states the predicate is "true for a blocked stop at Parent-Align, Finalize or the retired Merge-In name". The card carries a settled Decision, and a finding would overturn it with no `Added after Discussion` entry.
- guard-reach finding 2, the per-row assertion that no `overlap-note.md` exists guards a state that cannot occur: the acceptance criterion "A darn Finalize ... writes no overlap note" asks for it, and the assertion is cheap. It fixes the criterion in place, so the plan follows the answer key.
- guard-reach's out-of-lens remark, that the status-built moved stop's saved envelope has an empty `reason`: no lens raised it as a finding. D4 and card 3 already have that stop read by its `Error`, and the acceptance criterion names the status-built stop going to the arbiter, so the plan carries it.

## Focus verdicts

- Card order: card 3 and card 4 use `EscalateHalt` and `EscalateStopPrefix`, both of which card 2 creates. Neither card 2 nor card 3 reads card 4's `restep_hold`. The order holds, and this is not a defect.
- Fidelity: D1–D5, the Constraints and the PATTERN change are each carried by some card. The only departures from the decision record are F5's added clause and the doc gaps in F1–F4.
- Attack surface and refusal reach: card 1's removed guard and card 4's `restep_hold` skip each state what they let through and what bounds it, as do card 2's and card 3's routing edges. Card 2 names a reaching test for each escalate halt it prescribes: `TestFinalize_GateAndFixer`, `TestMergeIn_Fixer` and `TestFixerRun_Outcomes`.
- Edit card shape: `haltReason` carries the re-sign arrow. `ArbiterSeam.JudgesFirst` is a struct field and takes no arrow, so card 3 is correct as written.
