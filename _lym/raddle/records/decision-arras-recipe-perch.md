## Goal

The arras recipe and its split perch

Takes: raddle-perch-mode's `perch` mode and outcome checks; the motif and raddle recipes and the generic Bouncer, BurlerReview and BurlerFix engines
Delivers: `contracts/recipes/arras-recipe.yaml` registered wherever motif and raddle are, with its profile blocks and the test pinning `arras` as a former name reversed; the split perch segment with its loom keys, guard-category auto-accept key and the new rubric stencil `loom-rubric-raddle-review.md`; the perch re-arm on a fresh entry into Raddle-Write over a settled verdict; the run-level tests of an arras run through Discussion, the split perch and Raddle-File
Exports: the `arras` recipe name in shedrun, the split perch's loom keys

## Scope

The part changes these packages:

- contracts/recipes (about 140 changed lines)
- contracts/stencils (about 120 changed lines)
- internal/shedrun (about 15 changed lines)
- internal/loomrecipe (about 90 changed lines)
- internal/loomengine (about 150 changed lines)
- internal/loomcli (about 150 changed lines)
- internal/loomshed (about 40 changed lines)
- internal/shedrecipe (about 30 changed lines)
- internal/shedcli (about 20 changed lines)
- internal/battencli (about 40 changed lines)
- internal/runprofile (about 30 changed lines)
- internal/configsync (about 20 changed lines)
- internal/reviewbudget (about 20 changed lines)

## Decisions

### D1 — The `arras` recipe is the Motif block followed by raddle's write and file rows and the split perch

Decision: `contracts/recipes/arras-recipe.yaml` (new) declares `contract: loom` in its record-only form.
Its rows, in on_done order:

1. A block row `Motif` with `block: motif`, `block_gate: undiscussed` and `on_done: Raddle-Write`.
2. `Raddle-Write`, engine `RaddleWrite`, with its `raddle` gate.
3. The split perch of D2.
4. `Raddle-File`, engine `RaddleFile`, its `on_stuck` returning to `Raddle-Write` as in `contracts/recipes/raddle-recipe.yaml`.

It holds no `Record-Write` row: the Motif block's `DiscussionWrite` is the one record row the loom contract allows, and Raddle-Write reads the same decision record path either way.
The `undiscussed` gate seeds the stored record and skips Discussion when the entry is already discussed, so an arras run restarted after its Discussion approval resumes at Raddle-Write.
`arras` is registered wherever `motif` and `raddle` are, and the test case that pins `arras` as an unknown former recipe name is reversed.

Rationale: the Motif block is already how tapestry runs Discussion, so arras reuses both sub-flows without copying rows, and the loom contract needs no change.

### D2 — The split perch is a Bouncer ring over the proposal, with a cold fixer

Decision: three rows in a review segment of their own, `Raddle-Bouncer`, `Raddle-Burler-Review` and `Raddle-Burler-Fix`, built on the generic `Bouncer`, `BurlerReview` and `BurlerFix` engines.

- The artifact is `_lym/raddle/proposal.yaml`; the decision record is read-only context named in the rubric.
- The rubric is the new stencil `contracts/stencils/loom/loom-rubric-raddle-review.md` (new), embedded and registered like the other loom rubrics.
- The fixer is a cold `BurlerFix` session with `fix-scope: overlay` and the `raddle` gate, so a fixed proposal still passes every proposal check before the next round.
- The Bouncer sets no `carry_over`, so no round writes into the decision record, and no `approve_seam`, so the Discussion approval stays the recipe's one approval row.
- The Bouncer sets no `commit_seam`: Raddle-File's commit of the raddle directory commits the approved proposal.
- `escalate_decisions` is on, so decision-class findings go to the run's parent through the Bouncer, where the arbiter settles them first under its existing rules; the fixer row sets `decisions_escalated`.
- The segment takes its own loom keys, in the shape the Discussion segment's keys take: a bounce budget, reviewer and fixer fans, an auto-accept setting and reviewer, fixer and judge model keys.
  The fixer's model key defaults to the raddle key's model, so the split is fixed by a model of the raddle agent's strength.
  The split perch's auto-accept key, `<perch>_auto_accept` under its perch name, defaults to false and is a guard-category key, as `reviewbudget` registers every perch's auto-accept key.
- A fresh entry into Raddle-Write, a return from Raddle-File or a goto into the row, re-arms the perch when the perch's last verdict settled,
  as after a Raddle-File return or a goto from Raddle-File.
  The split perch's Bouncer has no Subject seam, so it keeps no settle or generation record,
  and its next entry over that settled verdict clears its run directory.
  The new proposal is then reviewed in a fresh cycle with the segment's full budget.
  Raddle-File's halt on its third failure bounds the cycles a run can spend.
- A goto into Raddle-Write from a perch that has not settled, such as one awaiting its parent on an escalation, opens no fresh cycle,
  and that is accepted.
  The Bouncer finds its recorded unsettled verdict, so it clears nothing and settles that round again, raising its escalation again.
  Shed's goto keeps the spent bounce count, and with no Subject seam no seed starts a new generation.
  Below the hard cap, `lym loom circling continue` runs the next round over the new proposal.
  At or past the hard cap the round can only be accepted,
  so `lym loom circling accept` files the new proposal with no round having reviewed it, as an accept below the cap does too.

Rationale: the Bouncer ring is the one perch mechanism the arbiter already serves, and a cold fixer reads only the proposal, the record and the findings.

### D3 — The split rubric judges the split against the whole record

Decision: the rubric directs the reviewer to read the decision record, every `### Added after Discussion` entry included, and to judge the split's judgement, not its format:

- Every decision, Added after Discussion entry included, lands in a part whose scope and contract can build it.
- The seams: each part's contract (takes, delivers, exports) matches what the parts on its `depends_on` path deliver.
- The order follows the record's stated order, or the departure is listed in `order_departures` with its cost.
- Each part's recipe and profile fit its size and risk, and the cost lines rest on the cells they cite.
- One task or a family fits the record's scope.

The mechanical checks of `contracts/specs/raddle-proposal-spec.md`, D5's outcome checks included, are named as already enforced by the gate and are not re-derived.

Rationale: an Added after Discussion entry carries no decision id, so the mechanical checks cannot tell whether the split builds it.

## Constraints

- `PATTERN-shed-recipe-registry`: arras declares `contract: loom`, and a verb that serves some loom recipes refuses on others by the recipe's rows, never its name (D4's mode, D7's refusal).
- `PATTERN-stencil-ownership`: the new rubric and the raddle stencil's mode text are read at call time through `stencilstore`.
- `PATTERN-sole-parsers`: D5's outcome checks live in `internal/raddleparser`, the proposal's sole parser, which takes the mode as an option.
- `PATTERN-refusal-way-forward`: the refusals of D4, D5, D7 and D8 name their way forward, and a test reaches each.
- `PATTERN-run-records`: post-Discussion decisions are written only by `lym loom decision add`.
- `PATTERN-dobby-runs`: released members start only through their parent's dobby, which D8 relies on and does not change.

## Auto-mode assumptions

- The perch's fixer is cold, not Raddle-Write re-entered (D2).
- A project release releases its draft members at once, and the project keeps the whole record (D8).
- Shape `beam` is refused in an arras run (D5).
- The family's project is the arras entry turned into a project (D6).

## Open risks

- A decision added after the perch's last approving round and before Raddle-File reaches every part's record but is not reviewed against the split's seams.

## Acceptance criteria

- D1: an `arras` task runs Discussion, its perch, Raddle-Write, the split perch and Raddle-File in that order, and the loader accepts the recipe under the loom contract.
- D1: an arras run restarted after its Discussion approval skips Discussion and resumes at Raddle-Write with the stored record.
- D1: `arras` is accepted as a board task's recipe and as a task worktree's run recipe.
- D2: the split perch reviews `_lym/raddle/proposal.yaml`, a fixed proposal that fails the `raddle` gate is not handed to the next round, and decision-class findings reach the run's parent through the Bouncer.
- D2: no split perch round changes the decision record, and `lym loom decision add` still finds the Discussion approval row in an arras run.
- D2: after a Raddle-File return or a goto to Raddle-Write from Raddle-File, the new proposal is reviewed by the perch in a fresh cycle with the segment's full budget before Raddle-File runs.
- D2: after a goto to Raddle-Write from a perch awaiting its parent on an escalation below the hard cap, the perch raises that escalation again, and `lym loom circling continue` then runs the next round over the new proposal.
- D2: with the split perch's auto-accept key off, the Bouncer never settles as an accept a spent-budget round holding an open BLOCKING or MEDIUM ledger entry not tagged `mechanical`: it continues automatically or escalates to the run's parent.
- Review: D3 — the rubric names the decision record and its Added after Discussion entries as the authority and names each judgement D3 lists.
- D4: an arras run records `perch` mode and reaches Raddle-File on the perch's approval with `raddle_auto_approve` off, and `lym loom raddle-approve` on it refuses naming the perch.
- D4: an arras run's Raddle-File finds a stored cold proposal after the perch's approval, also after a return.
- D4: with the split perch's auto-accept key off, a spent-budget round whose open BLOCKING and MEDIUM entries are all tagged `mechanical`, beside an open LOW entry of another tag, is accepted automatically, its decision file marked automatic, and Raddle-File files its proposal.
- D4: after `lym loom goto --to Raddle-Write` on an arras run, the last proposal is moved aside and the seat starts a new session.
- D5: a one-task arras run files one draft task with its record and leaves the arras entry done.
- D7: `lym loom decision add --by operator` during an arras run's Raddle-Write row appends the entry, stores the record on the board, and every filed part's record carries it.
