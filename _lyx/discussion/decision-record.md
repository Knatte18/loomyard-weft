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
- The orch role stencil's escalation guidance (`circling continue` "to grant more", and the small-task `review_max_bounces: 2` advice and its reason), updated for the hard cap and the rule.
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
Rounds below it run as today.
From the round at which the budget is reached, a `CONTINUE` or `CIRCLING` verdict on a round with no decision file and no escalation record is settled by this rule.
An entry counts as design-wrong when it is open, tagged `design-wrong` (D1), and BLOCKING or MEDIUM; a LOW or NIT entry never counts.
In order:

1. The facts carry a parse-error row for the round, or the round's ledger or (past the soft cap) the previous round's ledger cannot be read or parsed: escalate with cause `budget`.
2. No entry counts as design-wrong: accept unconverged, through the existing accept path (carry-over into `## Open risks`, approve, commit), recorded as an automatic accept.
3. The round is at or past the hard cap: escalate with cause `hard-cap`.
4. The verdict is an earned `CIRCLING`, or the round is past the soft cap and a design-wrong key was also open in the previous round's ledger: escalate with cause `circling`.
5. Otherwise run one more round: record an automatic `continue` decision for the round and return a budget-exempt Stuck.

The rule reads only on-disk state, so a harvest and a replay of the same round settle the same way, and its accept or continue is written as the round's decision file before the Bouncer returns.
A round with a decision file is acted on as today, except a `continue` at or past the hard cap, which escalates with cause `hard-cap` instead of running another round.
A round with an escalation record keeps it.

Bound: the rule accepts fixes no reviewer has seen only when no open BLOCKING or MEDIUM finding is tagged `design-wrong`, and it can buy rounds past the soft cap one at a time, never at or past the hard cap.

Rationale: the operator's soft-cap rule from the board entry, made mechanical in Go so the judge's only new job is the tag, and so neither a verdict nor a stale decision file can pass the hard cap.
"Blocker" in the board entry reads as a gating severity, so LOW and NIT findings never buy rounds or escalations.
An earned `CIRCLING` keeps escalating from the soft-cap round on, as it does below it.

### D3 — A hard cap per perch, never passed

Decision: loom config gains `review_hard_max_bounces`, default 4, below 1 refused, beside `review_max_bounces`.
Per-perch overrides `discussion_hard_max_bounces`, `plan_hard_max_bounces`, `webster_hard_max_bounces` and `merge_hard_max_bounces` take the run-wide value at 0 and are refused when negative, as the `<perch>_max_bounces` keys are.
The loom keys take run-settings category `none` in `loomengine`'s config spec, as the `*_max_bounces` keys do.
darn config gains `review_hard_max_bounces`, default 4, below 1 refused, for the Darn-Review perch, with the same layer policy as darn's `review_max_bounces` under `PATTERN-hub-wide-config`.
A perch whose resolved hard cap is below its resolved soft cap is refused at load, naming both keys; darn's soft cap of 0 skips the review, so the check applies there only to a soft cap above 0.
A hard cap equal to the soft cap is legal and means no round past the soft cap.
Each refusal's way forward is to raise the hard cap or lower the soft cap with `lyx config loom --set …` or `lyx config darn --set …` from the prime, then `lyx loom resume`; its spec row sits beside the existing loom "review key below 1" row or the darn `review_max_bounces` row.
The hard cap counts judged rounds of the segment's generation, as the soft cap's "N gives N judged rounds" does, and is enforced by the Bouncer and by `circling continue` (D5), both resolving it from the same config load at call time.

Rationale: the operator settled the hard cap's default, its own key and the per-perch pattern; darn has its own budget key, so its hard cap sits beside it.
Refusing hard below soft keeps the caps meaningful; equal caps let a hub turn the extra rounds off.

### D4 — Continues past the soft cap are budget-exempt

Decision: a `continue` decision on a round at or past the soft cap and below the hard cap returns a budget-exempt Stuck whatever its cause, whether the rule (D2) or a parent's or arbiter's `circling continue` wrote it.
Below the soft cap, a circling-cause continue stays counted as today.

Bound: an exempt continue skips Shed's generic bounce-budget block; the hard cap (D2, D3, D5) bounds it instead.

Rationale: past the soft cap Shed's count already equals its budget, so a counted Stuck would block the run where the rule or the arbiter chose one more round.

### D5 — `circling continue` refuses at the hard cap

Decision: `lyx loom circling continue` refuses on a round at or past its perch's hard cap, for every caller, the arbiter included.
The refusal's way forward is `lyx loom circling accept <slug>`, or raising the perch's hard cap as D3 names and then `lyx loom circling continue <slug>`.
It lands with its refusal-spec row and a test.

Bound: this adds a guard and removes none.

Rationale: the operator's "never exceeded" for the hard cap; without the refusal an arbiter or parent `continue` at a `hard-cap` escalation would run round H+1.

### D6 — `hard-cap` is a third escalation cause

Decision: the escalation record, the circling decision file (which copies its escalation's cause), the parent notice and the escalation brief carry cause `hard-cap` beside `circling` and `budget`, and their writers and readers accept all three.
At `hard-cap` the Awaiting Reason offers only `lyx loom circling accept`, and the brief stencil says the round can only be accepted or handed on.
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
- A4 → D2: a returning blocker is a design-wrong key open in the previous round too, past the soft cap; an earned `CIRCLING` escalates from the soft cap on.
- A5 → D3: darn gets the rule and its own hard-cap key.
- A6 → D3: a hard cap below the soft cap is refused, not raised.
- A7 → D8: auto-accept keeps its harvest meaning; its replay and `blocking_escalates` paths go to the rule.
- A8 → Scope: the arbiter's eligibility list is unchanged.

## Open risks

- The judge may tag a design-wrong finding `mechanical`, and the rule then accepts a fix nobody reviewed.
- A hub or task `run_settings` with a soft cap above 4 and no hard-cap key is refused at load once this lands, which parks a run in flight until the hard cap is raised (D3's way forward).
- An older `lyx` binary, such as prod during a deploy window, refuses a `hard-cap` escalation record or decision file it reads.
- A run already awaiting a `budget` escalation when the binary changes keeps that escalation.

## Acceptance criteria

- At the soft cap, a round with no open BLOCKING or MEDIUM `design-wrong` entry is accepted, carried over and committed without an escalation record or parent notice; an open LOW `design-wrong` entry does not prevent it.
- At the soft cap, a round with an open BLOCKING or MEDIUM `design-wrong` entry runs one more round, recorded as an automatic continue, without Shed's bounce-budget block.
- An earned `CIRCLING` with a design-wrong entry open escalates with cause `circling` at the soft-cap round and past it.
- Past the soft cap, a design-wrong key open in the previous round and again in this one escalates with cause `circling`.
- At the hard cap, a design-wrong entry escalates with cause `hard-cap`; with none, the round is accepted.
- A hard cap equal to the soft cap escalates `hard-cap` at the soft-cap round when a design-wrong entry is open.
- A recorded `continue` decision at or past the hard cap escalates with cause `hard-cap` instead of running another round.
- A ledger entry with no `fix` key reads as `design-wrong`.
- A parse-error facts row for the round, an unreadable round ledger and, past the soft cap, an unreadable previous ledger each escalate with cause `budget`.
- A replay of a round the rule settled acts on the recorded decision, and a replay of an unsettled round settles as the harvest would, auto-accept perches included.
- A non-Merge auto-accept perch still accepts a harvested round at the soft cap with a design-wrong entry open.
- A `blocking_escalates` row with an open BLOCKING entry is settled by the rule, not escalated.
- `lyx loom circling continue` is refused at or past the hard cap with the named way forward, including at a `hard-cap` escalation, and allowed on a `circling` escalation below it.
- A parent's or arbiter's continue past the soft cap is budget-exempt.
- A `hard-cap` escalation writes its record, brief and notice, reads back, and its Awaiting Reason offers only accept; its decision file reads back.
- Config load refuses `review_hard_max_bounces` below 1, a negative per-perch hard cap, and a resolved hard cap below a resolved soft cap, in loom and darn config, and the loom hard-cap keys are settable through `run_settings`.
- The judge, escalation-brief, arbiter and orch stencils render with the new wording.

## Notes for the plan writer

- The current budget path is `settleUndecided` and `settleUnconverged` in `internal/shedadapters/bouncer.go`; automatic accepts are recorded through `recordAutoAccept` in `circling.go`, and the decision file already carries an `automatic` flag.
- The Bouncer's escalation and circling test tables (`bouncer_escalation_test.go`, `bouncer_circling_test.go`) are the likely homes for the rule's cases.
- The loom keys and their run-settings categories sit in `internal/loomengine/config.go` and `template.yaml`; the per-perch maps are `perchMaxBounces` and `perchAutoAccept` in `internal/loomcli/wiring.go`, and the darn budget is wired in `internal/loomcli/darnwiring.go`.
- The escalation cause is validated in `internal/shedadapters/escalation.go` on write and read, and the decision file's cause in `circling.go`.
- The orch guidance is in `contracts/stencils/orch/orch-template-role.md`.
