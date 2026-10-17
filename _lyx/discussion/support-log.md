# Support log — pr-gate-run-identity

## Interview

Autonomous run: no operator answered; every pick below is the writer's own and is listed in the question ledger.

### Turn 1 — scope of issue #692

Read the board entry, the dependent entry `landing-trailer-text` and the board list.
`landing-trailer-text` depends on this task for ordering in the shared `internal/landingshed` package: it changes the landing config and Finalize's commit trailer, and this task changes Finalize's pull-request close, so no decision of either touches the other's surface.
The batten refusal the issue reports first already refuses with a way forward (`lyx batten cleanup --apply`, delete the task branch), which matches the issue's second accepted expectation, so it stays out.
The pull-request adoption is the fault to fix.

### Turn 2 — how a run recognises its own pull request

Read `FindPullRequest` and its callers: Publish, PR-Gate, PR-Align, Finalize's close, and the approve and reject verbs' `findPR` seams.
Two candidates: record the created number in run state, or bound the lookup by the run's start.
Picked the start bound from the pair's seed `started_at`, which `internal/hubgeom` already treats as the pair's start time.

### Turn 3 — an older open pull request

An older open pull request (an earlier Finalize's close failed, which only warns) would make GitHub refuse the create.
Picked a Stuck stop naming it and the way forward.

### Turn 4 — what counts as a decision note

Read `alignmentSnapshot`, the orch role stencil's decision-note rule and the bouncer escalation stencil's definition of a recorded operator decision.
Picked the orch's recommended fixed title form `Decision for <slug>:`, declared in `internal/alignment` and rendered into the orch role stencil through a template key.

### Turn 5 — the duplicate reason constant

Both declarations share `internal/alignment` as an import; picked one exported constant there.

### Turn 6 — lens review

Every fork returned a report.

- goal-scope, D2 reads as unlicensed scope: fixed, D2's rationale now says D1 makes the stop necessary.
- goal-scope, bounding every caller is unexplained: fixed, Scope says it is deliberate.
- goal-scope, premises, leanness and scenarios, seed freshness and its risk: merged and fixed; the assumption now names cleanup's removal of the run directory and `WriteSeed` keeping an existing seed, and one open risk covers both a relaxed batten refusal and a mid-run seed reset.
- goal-scope, existing free-form decision notes: fixed, D3's bound says they stop counting at landing.
- goal-scope, board refusal ruled out only in the support log: fixed, added to Scope Out.
- board-fit, `landing-trailer-text` overlap unstated: fixed in Turn 1 of this log; rejected for the record, since the plan writer gains nothing from it.
- board-fit, `shedrecipe-capabilities` Env threading: fixed by naming `landingshed.Deps` in D1.
- board-fit, `run-verbs-shared` and `orch-split`: rejected, informational with no conflict.
- board-fit, two meanings of decision note: rejected, Scope Out already separates the arbiter's definition.
- leanness, D1 rationale argues the rejected alternative: fixed, trimmed to positive facts.
- leanness, premises and scenarios, D2's implementation sentence and `FindPullRequest`'s return shape: merged and fixed, moved to Notes for the plan writer as the return-shape hint.
- leanness, `PATTERN-github-auth` and docs-rule constraints: fixed, cut.
- leanness, test-budget note: fixed, cut.
- leanness, Scope Out line on the board entry: fixed, cut.
- leanness, Scope and Notes overlap on `doc.go`: rejected, Notes carries the wording to change and Scope the package list.
- pattern-fit, `PATTERN-refusal-way-forward` does not bind `internal/landingshed`: fixed, Constraints says D2 follows its form by choice with the `way forward:` clause.
- pattern-fit, no PATTERN entry for the title form: rejected, Q7 now gives the reason.
- pattern-fit, `PATTERN-producer-pointer-rule` departure: fixed, the stencil renders the title form through a template key, so no second literal copy exists and no departure remains.
- premises, D2 rationale names only a failed Finalize close: fixed, an abandoned run added.
- scenarios, PR-Gate on an older merged pull request and PR-Align on an older open one: fixed, acceptance criteria added.
- scenarios, a seed that exists but cannot be read: fixed, D1 fails the wiring and the verbs refuse, with an acceptance criterion.
- scenarios, Finalize and an older open pull request: fixed, acceptance criterion added.
- scenarios, whether the entry's kind matters: fixed, D3 keeps kind note.
- scenarios, pruning a decision note mid-run escalates: rejected, that is the safe direction.

## Rejected alternatives

- Record the created pull request's number in run state and look it up by number: a crash between the create and the write leaves an orphan the next Publish cannot create past, and a pull request the orch opens by hand (the #691 workaround) would not be the run's.
- Look only at open pull requests: an earlier run's pull request left open by a failed Finalize close would still be adopted, and a human close of this run's own pull request would read as "none" and Publish would open a new one, hiding the human's stop.
- Refuse the slug at the board on reuse: the board does not know whether an earlier run landed, and batten already refuses a completed run's record.
- Decision notes recognised by a label: labels on this board name modules and types, and a note's title is what the orch already writes and the operator reads.
- Decision notes recognised by a body marker: the body is free text a mention can also contain, the same failure as today.
- Keep the reason constant in landingshed and import it from loomcli: it belongs to the snapshot check `internal/alignment` owns.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): Is batten's completed-record refusal in scope? No; it already names its way forward.
- Q2 (auto-pick): How does a run identify its pull request? By `created_at` not before the seed's `started_at`.
- Q3 (auto-pick): What happens with a seed lacking a stamp? Zero bound, today's behaviour, logged warning; a seed that exists but cannot be read fails the wiring instead.
- Q4 (auto-pick): What does Publish do with an older open pull request? Stop Stuck naming it, close and resume.
- Q5 (auto-pick): How is a decision note recognised? Title prefix `Decision for <slug>:`, case-sensitive, declared in `internal/alignment`.
- Q6 (auto-pick): Where does `decisionRecordedReason` live? One exported constant in `internal/alignment`.
- Q7 (auto-pick): Is a new PATTERN entry needed? No; the lookup already has one function every caller uses, and the decision-note title form has one Go declaration that the orch role stencil renders through a required template key, so no second copy exists for an entry to guard.
