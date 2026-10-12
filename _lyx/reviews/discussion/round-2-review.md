---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D2 (rule window, step 3) and D3 (hard cap never passed)
    summary: The hard cap is checked only inside the rule, whose window opens on Shed's counted Stucks, so after a goto resets that count a round at or past the hard cap runs as today and the run passes round H.
    fix: In D2, open the rule's window at whichever comes first, the soft cap by D2's bounce-budget measure or a round numbered at or past the hard cap by D3's round number, and say so in the paragraph that now reads "Rounds below it run as today"; state in D3 that the hard cap is enforced whatever Shed's count reads, naming the goto that resets the episode while the run directory's round numbers carry on. Add an Acceptance criterion - after a goto into the segment resets its bounce count, a CONTINUE round at the hard cap with a design-wrong entry open escalates with cause hard-cap, and with none it is accepted.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D3 (refusal way forward), D5 (way forward), D9 (run_settings advice)
    summary: The way forward to raise a hard cap is lyx config loom --set from the prime, which writes the default layer, but a hard cap set in a task's run_settings sits in the task's local layer above it, so the named way forward cannot work for the very setting D9 advises.
    fix: In D3 and D5, make the way forward name the layer the resolved key comes from - for a value from the task's run_settings, correct the task's run_settings with `lyx board upsert` and then `lyx batten run <slug>`, since Seed-Child rewrites the local layer from the board on every call; for a hub value, `lyx config loom --set …` or `lyx config darn --set …` from the prime as now. Add an Acceptance criterion that the circling continue refusal on a task whose hard cap comes from run_settings names the run_settings route.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2 (recorded continue at the hard cap) and Acceptance criteria
    summary: D2 says the Bouncer writes a hard-cap escalation record over a recorded continue at the hard cap, yet also says a round with an escalation record keeps it, and a parent's or arbiter's continue always answers an existing record, so the cause, the brief and the acceptance criterion's hard-cap cause disagree.
    fix: In D2, state that on a recorded continue at or past the hard cap the Bouncer moves both the continue decision file and the round's existing escalation record, if any, aside under retired names the readers ignore, then writes a fresh escalation record and brief with cause hard-cap; scope the sentence "A round with an escalation record keeps it" to rounds without a recorded continue at the hard cap. Keep the replay sentence, which then holds for both moves.
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 Bound
    summary: The Bound for an unreviewed mechanical-tagged fix leans on the accept path carrying open MEDIUM-or-worse entries into Open risks, but the Darn-Review perch has no carry-over, so on darn a mis-tagged design change is accepted with no such record.
    fix: Rewrite D2's Bound so the carry-over bound is stated for the loom perches only, into the decision record's Open risks section, and state for the Darn-Review perch what bounds it instead - the soft-cap window, the decision file's automatic flag, and the operator's review at PR-Gate.
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Acceptance criteria (earned CIRCLING at the soft cap) vs D8
    summary: The criterion that an earned CIRCLING with a design-wrong entry open escalates with cause circling at the soft cap is unqualified, but D8 has an auto-accept perch accept that harvested round without the rule, Merge-Review by default.
    fix: Qualify the criterion to a perch without auto-accept, or a replay, or a blocking_escalates round with an open BLOCKING entry.
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 step 4
    summary: Step 4 says the judge's earned CIRCLING already covers a key open in any earlier round, but CIRCLING is earned only from review_circling_checkpoint on, and the soft cap can sit below it, as Merge-Review's budget of 1 against a checkpoint of 3 does.
    fix: In D2 step 4, qualify the sentence - the earned CIRCLING covers a key open in an earlier round from review_circling_checkpoint on; below it, a key that returns after skipping a round runs on until the hard cap stops it.
---

### [BLOCKING] The hard cap is passed after a goto resets Shed's count

**Location:** `_lyx/discussion/decision-record.md` D2, D3
**Issue:** D2 opens the rule's window only when the bounce-budget seam reports the counted Stucks at or above the budget, and says rounds below it "run as today"; the hard-cap check is step 3 inside that window.
D3 measures the hard cap by round number in the segment's current generation, and a generation ends only on a settled one (`settledGeneration` in `internal/shedadapters/bouncer.go`: a CONVERGED verdict or a settled accept).
A goto into the segment ends Shed's episode and resets the count (`episodeStuckCount` in `internal/shedengine/run.go`), but leaves the run directory and its round numbers in place, as `Call`'s doc says ("a segment re-entered after a `lyx loom goto` … halts Awaiting again on that round").
A goto is the way forward Shed itself names on "bounce budget exhausted", which a degrade or a rejudge at a round past the soft cap triggers, since both are counted Stucks.
After it, a round numbered at or past H judged CONTINUE reads as below the soft cap, takes the plain counted Stuck, and the next round runs, up to a fresh budget's worth of rounds past H.
This contradicts Scope ("a hard cap … that no round passes"), D3 ("no round after round H runs") and the Goal.
D5's verb refusal does not cover it, since no escalation or continue decision is involved.
On focus item 2: the soft cap and D4's exemption use one measure, the counted Stucks, and the hard cap uses round numbers.
The two disagree through counted degrades and rejudges, which D2 and D3 state, and through a goto's count reset, which they do not.

### [MEDIUM] The hard-cap way forward misses a task's run_settings layer

**Location:** `_lyx/discussion/decision-record.md` D3, D5, D9
**Issue:** D3 makes the loom hard-cap keys run-settings category `none`, so a task's `run_settings` may set them, and D9 advises exactly that for small tasks (`review_hard_max_bounces: 3`).
Batten's Seed-Child writes the task's run settings into the task worktree's local config layer (`internal/battenshed/doc.go`), which stacks above the repo default that `lyx config loom --set` from the prime writes (`internal/configcli/doc.go`).
D5's refusal and D3's load refusals both name raising the hard cap "with `lyx config loom --set …` … from the prime", which leaves a local-layer value in force, so the named way forward does not unblock such a task, against `PATTERN-refusal-way-forward`.
A hand edit with `--layer local` would not hold either: Seed-Child discards it on the next resume.

### [MEDIUM] A recorded continue at the hard cap: new record or kept record

**Location:** `_lyx/discussion/decision-record.md` D2, Acceptance criteria
**Issue:** Focus item 4 asks whether a round holding a recorded continue can still be answered once it escalates at the hard cap.
The answer is yes: once the continue file is moved aside the round reads as pending, and `circlingVerb` (`internal/loomcli/circling.go`) records an accept over any pending escalation record.
But D2 says the Bouncer writes "that escalation record" with cause `hard-cap`, then says "A round with an escalation record keeps it", and `escalate` in `bouncer.go` rewrites nothing over an existing record.
A parent's or arbiter's continue is always the answer to an escalation, so such a round always holds a record, with cause `circling` or `budget` and a brief rendered when continue was still allowed.
Under the "keeps it" reading, the acceptance criterion's `hard-cap` cause fails and the stale brief contradicts D6's "only accepted or handed on".
Under the "writes" reading, D2 has to say what happens to the existing record.
Only a rule-written continue, which carries no record, matches D2's text as written.

### [MEDIUM] The mechanical-tag Bound does not hold on Darn-Review

**Location:** `_lyx/discussion/decision-record.md` D2 Bound
**Issue:** Focus item 5: D2 does state what a wrong `mechanical` tag lets through, an unreviewed design change.
Of what bounds it, one leg is that "the accept path carries every open MEDIUM-or-worse entry into the artifact's `## Open risks`".
The carry-over is wired only through the `carry_over` recipe key, which loom's Discussion, Plan, Webster and Merge Bouncer rows set (`contracts/recipes/loom-recipe.yaml`), while `contracts/recipes/darn-recipe.yaml` deliberately omits it ("a darn run has no decision record").
Scope puts Darn-Review under the rule, so on darn the stated bound is false, and the remaining bound is never named.
For loom's later perches the carry-over goes into the decision record (`writeCarryOver` in `internal/loomcli/wiring.go`), not into "the artifact" being reviewed.

### [LOW] Earned-CIRCLING criterion ignores the auto-accept harvest

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria, D8
**Issue:** D8 keeps an auto-accept perch's harvest accepting at the soft cap without the rule, and `settleUndecided` accepts there over a CIRCLING verdict too; `merge_auto_accept` defaults to true.
The criterion "An earned `CIRCLING` with a design-wrong entry open escalates with cause `circling` at the soft-cap round and past it" carries no such qualifier, so the two disagree on Merge-Review by default.
Focus item 1 is otherwise settled: D2's claim that a harvest and a replay settle alike is scoped to perches without auto-accept, and holds there, since both reach the rule over the same on-disk state.
On a non-Merge auto-accept perch with an open BLOCKING or MEDIUM design-wrong entry, D2, D8 and the criteria agree on one outcome per path: the harvest accepts, and a replay of the unsettled round continues under the rule (or escalates `hard-cap` at the hard cap).
That divergence is stated, not hidden, so it is not a defect.

### [LOW] Step 4's earned-CIRCLING premise needs the checkpoint

**Location:** `_lyx/discussion/decision-record.md` D2 step 4
**Issue:** `unearnedCircling` in `bouncer.go` reads a CIRCLING verdict as CONTINUE below `CirclingCheckpoint`.
`template.yaml` sets `review_circling_checkpoint: 3` and `merge_max_bounces: 1`, and a counted degrade can also bring the soft cap below the checkpoint.
Between the soft cap and the checkpoint, a design-wrong key open two rounds back, closed in the previous round and open again now is neither earned CIRCLING nor caught by step 4's previous-round check.
The hard cap bounds it, so the design holds, but the stated premise is wrong.
Focus item 3 is otherwise settled: the same key means the ledger `key`; the previous entry must itself count as design-wrong (open, BLOCKING or MEDIUM, tagged `design-wrong` or untagged); only the immediately previous round is consulted, after a continue on it.
A LOW, NIT or `mechanical` previous entry yields no return, so the round continues below the hard cap; an untagged one counts, so the round escalates `circling`.
