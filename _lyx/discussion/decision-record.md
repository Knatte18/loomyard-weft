# Decision record — judge-cap-call

## Goal

At a spent review budget, the Bouncer settles the round itself from the judge's per-finding tags instead of escalating to the parent and its arbiter.
An escalation remains only where a run is stuck (a design-wrong finding still open at a new hard cap, or a circling design-wrong finding from the soft cap on) or where the rule cannot read its inputs.

## Scope

In:

- The judge tags every open ledger entry `mechanical` or `design-wrong`, from the review text alone.
- The Bouncer's soft-cap rule, for every Bouncer row: loom's Discussion, Plan, Webster and Merge perches and darn's Darn-Review perch.
- A hard cap per perch, in loom config and in darn config, that no round passes.
- A third escalation cause, `hard-cap`, through the escalation record, the decision file, the Awaiting Reason and the brief.
- `lyx loom circling continue` refuses at the hard cap.
- The arbiter stencil's fixed review rule (`loom-template-arbiter.md`), rewritten for the causes it now gets.
  This takes over that one item of `arbiter-when`'s "Changes this implies"; that note keeps the eligibility list and the driver stencil.
- The orch role stencil's escalation guidance (`circling continue` "to grant more", and the small-task `review_max_bounces: 2` advice and its reason), updated for the hard cap and the rule (D9).
- Docs, template comments and refusal-spec rows for all of the above.

Out:

- The arbiter's eligibility list (`internal/loomcli/arbitereligibility.go`): it keeps admitting every pending Bouncer escalation, so `hard-cap`, `circling` and `budget` escalations reach the arbiter, and through it `go-driver-assistant`'s Go driver, unchanged; narrowing it is `arbiter-when`'s.
- The driver stencil's arbiter routing, the Go driver and the tackler (`go-driver-assistant`).
- The per-perch `*_auto_accept` flags' meaning on a harvested round.
- `lyx report convergence` and `report-timing` counting rule-made decisions.

## Decisions

### D1 — The judge tags every open ledger entry with a fix kind

Decision: every `status: open` ledger entry the judge writes carries `fix: mechanical` or `fix: design-wrong`, in every round.
`design-wrong` means the finding says the design itself was wrong, so its fix changed a decision, an approach, a contract or an interface, and that new text needs a fresh review.
`mechanical` means a local correction whose result the finding text already fixes, so the fix can be accepted unreviewed.
The tag is chosen from the review's finding text alone; the judge still never opens the artifacts or the fixer's diff.
The tag is asked for in every round, so the judge stencil needs no Go-held variant for it.
A ledger entry with no `fix` key, or an unknown value, reads as `design-wrong`.

Rationale: the call the arbiter made in every recorded budget escalation was this classification, and the judge already reads the findings that carry it.
Reading a missing tag as `design-wrong` errs toward one more review, bounded by the hard cap (D3).

### D2 — The soft-cap rule replaces the budget escalation

Decision: the soft cap is the perch's existing bounce budget (`review_max_bounces`, or the perch's `<perch>_max_bounces`, or darn's `review_max_bounces`).
A round is at or past the soft cap when the Bouncer's bounce-budget seam reports the segment's counted Stucks at or above its budget, as `budgetReached` compares today.
That count also holds the seed, every degrade and every rejudge, so the soft cap can be reached at a round numbered below the budget; this measure decides D4's exemption.
The rule's window opens at whichever comes first: the soft cap by this bounce-budget measure, or a round numbered at or past the hard cap by D3's round number.
Rounds before the window opens run as today.
The second leg matters after a goto into the segment, which resets Shed's count while the run directory's round numbers carry on: a round at or past the hard cap then reads below the soft cap, and the rule still settles it.
From the round at which the window opens, a `CONTINUE` or `CIRCLING` verdict on a round with no decision file and no escalation record is settled by this rule.
An entry counts as design-wrong when it is open, tagged `design-wrong` (D1, a missing or unknown tag included), and BLOCKING or MEDIUM; a LOW or NIT entry never counts.
In order:

1. The facts carry a parse-error row for the round, or the round's ledger cannot be read or parsed, or step 4 consults the previous round's ledger and it cannot be read or parsed: escalate with cause `budget`.
2. No entry counts as design-wrong: accept unconverged, through the existing accept path (carry-over into `## Open risks` where the recipe sets `carry_over`, approve, commit), recorded as an automatic accept.
3. The round is at or past the hard cap (D3): escalate with cause `hard-cap`.
4. The verdict is an earned `CIRCLING`, or the previous round's decision file records a `continue` and an entry that counts as design-wrong has the same ledger key as an entry that counts as design-wrong in the previous round's ledger: escalate with cause `circling`.
   Only the immediately previous round is consulted, and only after a continue was granted on it: from `review_circling_checkpoint` on, the judge's earned `CIRCLING` covers a key open in an earlier round, and this check adds only a blocker the granted round did not close.
   Below the checkpoint, which the soft cap can sit below (Merge-Review's budget of 1 against a checkpoint of 3), a key that returns after skipping a round runs on until the hard cap stops it.
   A key whose previous entry was LOW, NIT or tagged `mechanical` got no round for its design fix yet, so it does not return.
5. Otherwise run one more round: record an automatic `continue` decision for the round, with cause `budget`, and return a budget-exempt Stuck.

The rule reads only on-disk state, and its accept or continue is written as the round's decision file before the Bouncer returns.
A harvest and a replay of the same round settle the same way on a perch without auto-accept; on an auto-accept perch the harvest accepts and the replay goes to the rule (D8).
A round with a decision file is acted on as today, except a `continue` at or past the hard cap, which escalates with cause `hard-cap` instead of running another round.
A parent's or arbiter's continue always answers an escalation, so such a round holds an escalation record with cause `circling` or `budget` and a brief rendered while continue was still allowed.
On a recorded continue at or past the hard cap, the Bouncer moves both the `continue` decision file and the round's existing escalation record, if any, aside under retired names the decision and escalation readers ignore, then writes a fresh escalation record and brief with cause `hard-cap`.
The round then reads as pending, and `lyx loom circling accept` can record its decision.
A replay after the moves and before the fresh record is written finds no decision file and no escalation record, and the rule settles the round.
A round with an escalation record and no recorded continue at the hard cap keeps its record.

Bound: the rule accepts fixes no reviewer has seen only when no open BLOCKING or MEDIUM finding is tagged `design-wrong`, and it can buy rounds past the soft cap one at a time, never at or past the hard cap.
A finding the judge mis-tags `mechanical` lets an unreviewed design change into the artifact.
Nothing re-reviews that change.
On loom's perches, what bounds it is that the rule runs only at or past the soft cap, that the accept path carries every open MEDIUM-or-worse entry into the decision record's `## Open risks` section with its class and severity, and that the decision file marks the accept automatic.
On the Discussion, Plan and Webster perches, the segments downstream review their own artifacts built on it, which catches a wrong design only indirectly.
On the Merge-Review and Darn-Review perches, no later review segment reads the accepted artifact.
The only later gate is PR-Gate, where the alignment agent approves and lands the task with no human read while `alignment_pr_approve` is true, its default, and the parent decides only when it is false.
What bounds it there is the soft-cap window and the decision file's `automatic` flag, and on Merge-Review the `## Open risks` carry-over; Darn-Review has no carry-over, since a darn run has no decision record.

Rationale: the operator's soft-cap rule from the board entry, made mechanical in Go so the judge's only new job is the tag, and so neither a verdict nor a stale decision file can pass the hard cap.
"Blocker" in the board entry reads as a gating severity, so LOW and NIT findings never buy rounds or escalations.
An earned `CIRCLING` keeps escalating from the soft-cap round on, as it does below it.

### D3 — A hard cap per perch, never passed

Decision: loom config gains `review_hard_max_bounces`, default 4, below 1 refused, beside `review_max_bounces`.
Per-perch overrides `discussion_hard_max_bounces`, `plan_hard_max_bounces`, `webster_hard_max_bounces` and `merge_hard_max_bounces` take the run-wide value at 0 and are refused when negative, as the `<perch>_max_bounces` keys are.
The loom keys take run-settings category `none` in `loomengine`'s config spec, as the `*_max_bounces` keys do.
darn config gains `review_hard_max_bounces`, default 4, below 1 refused, for the Darn-Review perch, with the same layer policy as darn's `review_max_bounces` under `PATTERN-hub-wide-config`.
A perch whose resolved hard cap is below its resolved soft cap is refused at load, naming both keys; darn's soft cap of 0 skips the review, so the check applies there only to a soft cap above 0.
Each refusal's way forward is to raise the hard cap or lower the soft cap in the layer the resolved key comes from, written so it reaches the task worktree's own config, then `lyx loom resume`:

- A loom value from the task's `run_settings`, the local layer: `lyx config loom --layer local --set <key>=<n>` in the task worktree, and `lyx board upsert` correcting the task's `run_settings` too, so a later Seed-Child writes the same value.
  Seed-Child runs once, before the refusal can arise, so the board edit alone never reaches the running task.
- A loom value from the task worktree's tracked layer: `lyx config loom --set <key>=<n>` in the task worktree.
- A loom value from the hub default: `lyx config loom --layer default --set <key>=<n>`.
  Loom is not hub-wide, so `lyx config loom --set` from the prime writes the prime's tracked file, which an already forked task branch does not read.
- A darn value, which comes only from the hub file since darn's budget keys are barred from the local layer: `lyx config darn --set <key>=<n>` from the prime.
Its spec row sits beside the existing loom "review key below 1" row or the darn `review_max_bounces` row.
The hard cap is measured by the round's number, counted from 1 in the segment's current generation: a round numbered H or higher is at or past a hard cap of H, so the Bouncer never continues a round numbered H or higher, and such a round is accepted or escalated whatever produced it, a goto onto the segment's review row included.
Exempt continues never raise Shed's count, so the hard cap is never measured by it.
The hard cap is enforced whatever Shed's count reads: a `lyx loom goto` into the segment ends Shed's episode and resets its bounce count, while the run directory's round numbers carry on, so a round at or past the hard cap after a goto opens D2's window by its round number alone.
It is enforced by the Bouncer and by `circling continue` (D5), both resolving it from the same config load at call time.
A hard cap equal to the soft cap is legal: with no degrade or rejudge counted, the soft cap is reached at round H itself and the rule never continues; a counted degrade or rejudge can reach it earlier, and the rule may then continue up to round H.

Rationale: the operator settled the hard cap's default, its own key and the per-perch pattern; darn has its own budget key, so its hard cap sits beside it.
Refusing hard below soft keeps the caps meaningful; equal caps let a hub turn the extra rounds off.

### D4 — Continues past the soft cap are budget-exempt

Decision: a `continue` decision on a round at or past the soft cap, by D2's bounce-budget measure, and below the hard cap, by D3's round number, returns a budget-exempt Stuck whatever its cause, whether the rule (D2) or a parent's or arbiter's `circling continue` wrote it.
Below the soft cap, a circling-cause continue stays counted as today.

Bound: an exempt continue skips Shed's generic bounce-budget block; the hard cap (D2, D3, D5) bounds it instead.

Rationale: past the soft cap Shed's count already equals its budget, so a counted Stuck would block the run where the rule or the arbiter chose one more round.

### D5 — `circling continue` refuses at the hard cap

Decision: `lyx loom circling continue` refuses on a round at or past its perch's hard cap, for every caller, the arbiter included.
The refusal's way forward is `lyx loom circling accept <slug>`, or raising the perch's hard cap by D3's route for the layer its resolved value comes from, and then `lyx loom circling continue <slug>` in place of D3's `lyx loom resume`.
It lands with its refusal-spec row and a test.

Bound: this adds a guard and removes none.

Rationale: the operator's "never exceeded" for the hard cap; without the refusal an arbiter or parent `continue` at a `hard-cap` escalation would run round H+1.

### D6 — `hard-cap` is a third escalation cause

Decision: the escalation record, the circling decision file, the parent notice and the escalation brief carry cause `hard-cap` beside `circling` and `budget`, and their writers and readers accept all three.
A decision file answering an escalation copies its cause; a decision file the rule writes (D2) carries cause `budget` and the `automatic` flag.
On a round at or past its perch's hard cap, whatever the cause, the Awaiting Reason offers only `lyx loom circling accept`, and the brief stencil says the round can only be accepted or handed on.
The brief stencil explains each cause; `budget` now means the rule could not read its inputs.

Rationale: the arbiter and the parent need to tell a stuck run at the hard cap from a circling one, since only the latter may continue.

### D7 — The arbiter's review rule follows the causes

Decision: the arbiter stencil's fixed review rule ("accept unless a finding makes the design wrong, and hand on a blocker that remains after round 3") is replaced by a rule per cause, pointing at the judge stencil for what `design-wrong` and `mechanical` mean instead of restating them:

- `hard-cap`: read the open design-wrong findings against the code; accept when a finding is wrong or the fix in place answers it, otherwise hand on.
- `circling`: accept when the returned finding is wrong; continue when the fix is converging and the round is below the hard cap; otherwise hand on.
- `budget`: settle from the review, ledger and verdict files as for `circling`, or hand on.

A `hand-on` goes to the orch as today.
Whether `budget` stays arbiter-eligible is `arbiter-when`'s call; this rule covers it while it does.

Rationale: the old rule is now the judge's tag and Go's rule; the arbiter keeps the calls that need the code read.

### D8 — Auto-accept keeps its meaning on a harvest; the rule takes its escalations

Decision: a perch with `*_auto_accept` true still accepts a harvested round at the soft cap without the rule, as today.
Two of its paths that escalate with cause `budget` today go to the rule (D2) instead:

- a replay of a round with no decision file and no escalation record;
- on a Bouncer row whose recipe sets `blocking_escalates` (today only Merge-Bouncer), a round whose ledger holds an open BLOCKING entry.

An unreadable ledger still escalates with cause `budget`, through D2's first step.

Rationale: auto-accept is a per-perch operator choice this task does not reopen; its two escalation paths are the same budget stop this task moves off the arbiter.
A replay keeps today's exclusion from the auto-accept, so on these perches a replay of an unsettled round is settled by the rule rather than as its harvest would have been.

### D9 — The orch's escalation guidance follows the hard cap

Decision: the orch role stencil's guidance for a review loop that ran out of rounds says `lyx loom circling continue <slug>` grants one more round only below the perch's hard cap, and that at or past it, or at a `hard-cap` escalation, the way forward is `lyx loom circling accept <slug>` or raising the hard cap by D3's route for the layer its value comes from.
The small-task advice keeps `review_max_bounces: 2`, with its reason that a second round reviews the first round's fixes, and adds `review_hard_max_bounces: 3`, so the `run_settings` it advises read `{"loom": {"review_max_bounces": 2, "review_hard_max_bounces": 3}}`.

Rationale: the orch's verb now refuses at the hard cap (D5), so guidance offering it there would send the orch into a refusal.
A small task under the default hard cap of 4 could run twice its soft cap; a hard cap of 3 lets the rule buy one reviewed round for a design-wrong fix and no more.

### Added after Discussion (arbiter, 2026-10-12): Hub-default hard-cap route runs from the prime

Decision: D3's route for a loom value from the hub default, lyx config loom --layer default --set <key>=<n>, is run from the hub's prime worktree by the operator or the hub orch, and the refusal's way forward says so, as the darn route already does.

Rationale: configcli's setModule applies the hub write guard to every --layer default write, refusing it from a task session or a non-prime worktree, so a way forward that omits the prime would send a task session into that refusal.

## Constraints

- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: each new refusal (hard cap below 1 or negative, hard below soft, `circling continue` at the hard cap) names a way forward the agents may run and lands with a refusal-spec row and a test.
- `PATTERN-stencil-ownership`: the judge, escalation-brief, arbiter and orch wording lives in their stencils, read at call time; Go interpolates only values, and the judge stencil stays free of `{{if}}`/`{{range}}`.
- `PATTERN-producer-pointer-rule`: the arbiter stencil and the brief point at the judge stencil's tag meanings.
- `PATTERN-config-layers`, `PATTERN-config-strictness` and `PATTERN-hub-wide-config`: the new keys are declared in the loom and darn config templates and loaded strictly; darn's key follows its module's hub-wide layer policy.
- `PATTERN-test-speed` and `PATTERN-test-economy`: the rule is pure over on-disk files and tested at the unit tier.
- `PATTERN-documentation-lifecycle` and the CLAUDE.md docs rule: `shedadapters`' and `loomcli`'s `doc.go`, the config templates' comments and `contracts/specs/refusal-spec.md` change in the same commit.

## Auto-mode assumptions

- A1 → D1: the tag is asked for in every round.
- A2 → D1: a missing or unknown tag reads as `design-wrong`.
- A3 → D2: only BLOCKING and MEDIUM entries count as design-wrong.
- A4 → D2: a returning blocker is a key that counts as design-wrong in this round and in the previous round, after a continue granted on the previous round; an earned `CIRCLING` escalates from the soft cap on.
- A5 → D3: darn gets the rule and its own hard-cap key.
- A6 → D3: a hard cap below the soft cap is refused, not raised.
- A7 → D8: auto-accept keeps its harvest meaning; its replay and `blocking_escalates` paths go to the rule.
- A9 → D9: the small-task advice adds a hard cap of 3.
- A8 → Scope: the arbiter's eligibility list is unchanged.

## Open risks

- The judge may tag a design-wrong finding `mechanical`, and the rule then accepts a fix nobody reviewed, bounded only as D2's Bound says; on Merge-Review and Darn-Review no later review reads that fix, and by default PR-Gate lands it with no human read.
- A hub or task `run_settings` with a soft cap above 4 and no hard-cap key is refused at load once this lands, which parks a run in flight until the hard cap is raised or the soft cap lowered, by D3's route for the layer the value comes from.
- An older `lyx` binary, such as prod during a deploy window, refuses a `hard-cap` escalation record or decision file it reads.
- A run already awaiting a `budget` escalation when the binary changes keeps that escalation.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 3: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/discussion/round-3-review.md; fixer report: _lyx/reviews/discussion/round-3-fixer-report.md.
  - hard-cap-way-forward-misses-run-settings-layer (class design, severity BLOCKING, open in rounds 2, 3)
  - mechanical-mistag-bound-missing (class design, severity MEDIUM, open in rounds 1, 2, 3)
  - replay-vs-harvest-auto-accept-outcome (class consistency, severity MEDIUM, open in rounds 1, 2, 3)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- At the soft cap, a round with no open BLOCKING or MEDIUM `design-wrong` entry is accepted, carried over and committed without an escalation record or parent notice; an open LOW `design-wrong` entry does not prevent it.
- At the soft cap, a round with an open BLOCKING or MEDIUM `design-wrong` entry runs one more round, recorded as an automatic continue, without Shed's bounce-budget block, on a perch without auto-accept, on a replay, and on a `blocking_escalates` round with an open BLOCKING entry.
- An earned `CIRCLING` with a design-wrong entry open escalates with cause `circling` at the soft-cap round and past it, on a perch without auto-accept, on a replay, and on a `blocking_escalates` round with an open BLOCKING entry.
- After a continue decision on the previous round, a key that counts as design-wrong in the previous round and again in this one escalates with cause `circling`, on a perch without auto-accept, on a replay, and on a `blocking_escalates` round with an open BLOCKING entry.
- After a continue decision on the previous round, a key that was LOW or tagged `mechanical` there and counts as design-wrong now runs one more round below the hard cap, on a perch without auto-accept, on a replay, and on a `blocking_escalates` round with an open BLOCKING entry.
- The rule's automatic continue is recorded with cause `budget` and the `automatic` flag.
- With a counted degrade before it, the soft cap is reached at a round numbered below the budget, and, on a perch without auto-accept, on a replay, and on a `blocking_escalates` round with an open BLOCKING entry, the rule settles that round with its continue budget-exempt.
- At the hard cap, on a perch without auto-accept, on a replay, and on a `blocking_escalates` round with an open BLOCKING entry, a design-wrong entry escalates with cause `hard-cap`; with none, the round is accepted.
- After a goto into the segment resets its bounce count, a `CONTINUE` round at the hard cap with a design-wrong entry open escalates with cause `hard-cap`, and with none it is accepted.
- A hard cap equal to the soft cap escalates `hard-cap` at the soft-cap round when a design-wrong entry is open.
- A recorded `continue` decision at or past the hard cap is moved aside with the round's existing `circling` or `budget` escalation record, and the round escalates with a fresh record and brief of cause `hard-cap` instead of running another round; `lyx loom circling accept` then records its decision and the round is accepted.
- A ledger entry with no `fix` key reads as `design-wrong`.
- A parse-error facts row for the round, an unreadable round ledger and, after a continue decision on the previous round, an unreadable previous ledger each escalate with cause `budget`.
- A `budget` escalation on a round at or past the hard cap offers only `lyx loom circling accept` in its Awaiting Reason and brief.
- A replay of a round the rule settled acts on the recorded decision, and a replay of an unsettled round is settled by the rule on every perch, auto-accept perches included.
- A non-Merge auto-accept perch still accepts a harvested round at the soft cap with a design-wrong entry open.
- A `blocking_escalates` row with an open BLOCKING entry is settled by the rule, not escalated.
- `lyx loom circling continue` is refused at or past the hard cap with the named way forward, including at a `hard-cap` escalation, and allowed on a `circling` escalation below it.
- On a task whose hard cap comes from its `run_settings`, the `circling continue` refusal names `lyx config loom --layer local --set <key>=<n>` in the task worktree, with `lyx board upsert` correcting the task's `run_settings` too, and then `lyx loom circling continue <slug>`.
- A parent's or arbiter's continue past the soft cap is budget-exempt.
- A `hard-cap` escalation writes its record, brief and notice, reads back, and its Awaiting Reason offers only accept; its decision file reads back.
- Config load refuses `review_hard_max_bounces` below 1, a negative per-perch hard cap, and a resolved hard cap below a resolved soft cap, in loom and darn config, and the loom hard-cap keys are settable through `run_settings`.
- The judge, escalation-brief, arbiter and orch stencils render with the new wording; the orch stencil's small-task advice names `review_hard_max_bounces: 3`.

## Notes for the plan writer

- The current budget path is `settleUndecided` and `settleUnconverged` in `internal/shedadapters/bouncer.go`; automatic accepts are recorded through `recordAutoAccept` in `circling.go`, and the decision file already carries an `automatic` flag.
- The Bouncer's escalation and circling test tables (`bouncer_escalation_test.go`, `bouncer_circling_test.go`) are the likely homes for the rule's cases.
- The loom keys and their run-settings categories sit in `internal/loomengine/config.go` and `template.yaml`; the per-perch maps are `perchMaxBounces` and `perchAutoAccept` in `internal/loomcli/wiring.go`, and the darn budget is wired in `internal/loomcli/darnwiring.go`.
- The escalation cause is validated in `internal/shedadapters/escalation.go` on write and read, and the decision file's cause in `circling.go`.
- The orch guidance is in `contracts/stencils/orch/orch-template-role.md`.
