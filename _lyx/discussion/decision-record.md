# Decision record — loom-bouncer-rounds

## Goal

A Bouncer review loop never halts a run for a human on something the run's parent orchestrator can settle.
The judge rules `CIRCLING` only from a configured checkpoint round on, and only on unchanged or reopened finding keys, never on a rising count.
A `CIRCLING` ruling and a bounce budget spent without convergence both escalate to the parent, whose one-shot fork accepts or continues and records any design calls in the decision record through a new verb.
That verb is also how any scope addition made after Discussion enters the decision record, so plan review reads it as part of the answer key.
A recurring ledger finding is no longer filed as a `selfreport` GitHub issue.
`go test -tags smoke ./internal/loomcli/` passes.

## Scope

In:

- The Bouncer judge's ruling rules: checkpoint judging, the no-progress definition of `CIRCLING`, and a Go guard that downgrades an unearned `CIRCLING` to `CONTINUE`.
- Two new `loom.yaml` keys: the checkpoint round and the review segments' bounce budget.
- The review segments' bounce budget moves out of the recipe's review-segment rows into `loom.yaml`.
- Escalation of a `CIRCLING` ruling and of a spent review-segment budget to the parent: an escalation brief, a one-line parent notice carried on the `awaiting` envelope, and `ly-drive` relaying that notice to `LYX_PARENT`.
- Extending the existing `lyx loom circling accept|continue` verbs to settle a budget escalation as well as a `CIRCLING` one.
- A new verb, `lyx loom decision add`, that appends one decision to the decision record and re-runs the mechanical discussion check.
- Naming that verb in the Plan-Write stencil and the escalation brief, so post-Discussion additions go through it.
- Removing the `recurring-finding` anomaly trigger from `loomengine`'s detector and `loomcli`'s selfreport filing.
- Fixing the failing loomcli smoke tests.
- Docs: `internal/shedadapters/doc.go`, `internal/loomcli` docs, `docs/overview.md` (the circling and budget paragraphs), `contracts/specs/refusal-spec.md` rows for every new refusal.

Out:

- `internal/treadleengine`'s own `CIRCLING` judge: it has no consumer.
- The PR-Review segment (`PR-Gate`/`PR-Rework`): it has no Bouncer and keeps its recipe `max_bounces` and its blocking budget.
- Running the smoke tier inside the normal verify command; the `test-tiers` board note owns tier policy.
- Any change to the Discussion-Write parent-review gate (`internal/parentreview`).
- Batten's resume logic: the parent's fork resumes the run itself (D9).

## Decisions

### D1 — Checkpoint judging

Decision: Below the checkpoint round the judge may rule only `CONVERGED` or `CONTINUE`.
From the checkpoint round on, every judge call may also rule `CIRCLING`; with progress it rules `CONTINUE`, which by itself extends the loop only up to the review budget (D3); a round past the budget needs a recorded decision (D4).
Go renders the judge stencil's decision rule per round through a marker, one variant below the checkpoint without `CIRCLING` and one at or above it, so the stencil stays conditional-free.

Rationale: The operator's decision of 2026-10-05.
Applying the checkpoint test at every round from the checkpoint on, rather than at exactly one round, keeps a loop that stalls after the checkpoint escalable before the budget is spent.

### D2 — What counts as circling

Decision: `CIRCLING` means no progress: a gating finding (the existing gating definition: `design` class at MEDIUM or worse, or any BLOCKING) sits on a ledger key that is open in the round being judged and was open in an earlier round, either unchanged or reopened after being resolved.
A rising finding count alone, or new keys alone, is never circling.
The facts file Go renders gains, per recurring open key, the rounds it was open in, so the judge reads recurrence from the facts rather than inferring it.
Go computes each key's `rounds` from the ledgers of the earlier rounds; the judge's ledger for the judged round supplies only the keys, their status, class and severity.
Go guards the harvest: a `CIRCLING` verdict is read as `CONTINUE`, with a Warn naming the round and the reason, when the round is below the checkpoint, or when the ledger the judge wrote for that round carries no gating finding on an open key whose `rounds` include the judged round and at least one earlier round.

Bound: the guard only narrows `CIRCLING` to `CONTINUE`; it never produces `CIRCLING` or `CONVERGED`, and it never touches a `CONTINUE` or `CONVERGED` verdict.
A legacy verdict on disk keeps its existing reading.

Rationale: The `board-model` halt was judged on a rising count across fresh keys in a widened area; the operator ruled that disproportionate.
A Go check on the judge's own ledger makes the rule enforceable rather than prompt-only.

### D3 — Both numbers live in `loom.yaml`

Decision: `loom.yaml` gains two keys with defaults in its embedded template: `review_circling_checkpoint` (default 3) and `review_max_bounces` (default 5).
`review_max_bounces` is the bounce budget of every Bouncer review segment's rows (Discussion-Review, Plan-Review, Webster-Review); the recipe's review-segment rows drop their literal `max_bounces`, and the loom recipe assembly applies the config value to them.
`PR-Gate` keeps its recipe `max_bounces`.
Both keys must be positive integers; a value that is not is a config load error naming the key.
A checkpoint above the rounds the budget allows is accepted: such a run never rules `CIRCLING` and reaches the budget escalation (D4) instead.

Rationale: The operator called both numbers configuration; the recipe is embedded and needs a rebuild to tune, while `loom.yaml` is the run's tunable config and already carries the review roles' model specs.
One key per number, uniform across review segments, avoids designing per-segment tuning nobody asked for.

### D4 — A spent review budget escalates instead of blocking

Decision: When the judge rules `CONTINUE` and the `Bounces` seam reports the budget reached (below), the Bouncer returns `awaiting` with an escalation (D5) instead of the `Stuck` that Shed would turn into a `bounce-budget-exhausted` block.
The Bouncer reads the count through its existing `Bounces` seam; Shed's generic budget block is unchanged and stays the backstop for every other row and for a failed seam read.
"Reached" is the `Bounces` seam's own `count >= budget`: `count` is the Bouncer row's own counted `Stuck`s before this round's `Stuck`, the seed `Stuck` included, and `budget` is Shed's effective budget for that row; this is the `>=` Shed applies to the history it read before appending.
The seam's budget is the config-applied `review_max_bounces` (D3) that Shed's engine blocks on, never the bare embedded recipe, whose rows no longer carry `max_bounces`:
the routing projection that feeds the seam (`loomrecipe.Routing()` and `segmentBounces` in `internal/loomcli/wiring.go`) applies it through the assembly D3 uses for the engine, and a test pins that the seam and Shed agree.
No separate `review_max_bounces` comparison exists, so the Nth judged round of N escalates exactly where Shed would block.
The escalation cause is `budget` whenever the budget is reached in that sense, even when the judge also ruled `CIRCLING`; otherwise it is `circling`.
A `continue` on a circling escalation is an ordinary counted `Stuck`; when the `Bounces` seam returns `ok` true, it cannot hit Shed's block, because the round at which Shed would block has cause `budget`.
Every review-segment Bouncer is wired with the `Bounces` seam.
An unwired Bouncer, an `ok` false read or an errored read does not escalate on budget and falls to Shed's generic block.
The existing `circlingReason` clause that a continue will block on the budget is removed.
A `continue` decision on a budget escalation grants exactly one more round: the Bouncer's `Stuck` for that decision and the round producer's following `Stuck` are both `BudgetExempt`.
The next judge call finds the budget still spent and escalates again.

Bound: the exemption covers only the two `Stuck`s of the one round a recorded `continue` decision on a budget escalation names; it cannot be granted without a decision file for that round, and each further round needs its own decision.
No count caps the decisions; the committed decision files record each one (Open risks).
`lyx loom goto` still grants a fresh budget as today.

Rationale: The operator's decision: a budget spent without convergence messages the parent like a `CIRCLING` ruling.
Escalating from the Bouncer keeps `shedengine` generic, and one round per decision keeps the parent from opening an unbounded loop by one answer.

### D5 — Escalation artifacts

Decision: On an escalation (D1 `CIRCLING` or D4 budget) the Bouncer writes `round-<N>-escalation.md` into its run directory, rendered from a new Bouncer stencil, and returns `awaiting`.
The brief names: the segment, the round, the escalation cause (circling or budget), the latest review, the ledger, the verdict, the decision record, the task worktree path, and the commands the fork runs (D7, D9).
The `awaiting` output carries a one-line parent notice, rendered from a second new stencil, naming the slug and the brief path and asking the parent to review it with a one-shot fork.
Shed persists the notice on the status file and the step envelope as a new optional field (`parent_notice`), set only on `awaiting`.
The `awaiting` Reason keeps naming the `lyx loom circling` verbs and the resume, so a run with no parent still tells the operator the way forward.
Re-calling the Bouncer over the same on-disk state returns the same `awaiting` and rewrites nothing.
The brief and notice are durable run-directory files the wiring commits like the rest of the run directory.

Bound: `parent_notice` is advisory text; nothing in Go reads it back for control flow, and a missing or failed render degrades to the plain Reason with a Warn.
The notice interpolates only the slug and the brief path, never judge, review or ledger text, because the parent session acts on it as an instruction.

Rationale: The parent-review gate's shape (a brief plus a one-line message) is what the operator named; keeping the wording in stencils respects `PATTERN-stencil-ownership` and `PATTERN-shuttle-provider-seam`.

### D6 — Delivery through the driver

Decision: `ly-drive`, at an `awaiting` stop whose envelope carries a non-empty `parent_notice` and with `LYX_PARENT` set, sends that notice verbatim to `LYX_PARENT` instead of its generic stop-report line, records the send in the stop report, and hands back as today.
With `LYX_PARENT` unset or empty, or a failed send, the stop report and Reason are the escalation to the operator, as today.
The skill branches on the envelope field only, not on a row, recipe or driver value.

Rationale: The driver already notifies the parent at hand-backs; the Bouncer stays free of any driver or parent knowledge, which `PATTERN-driver-choice-single-site` requires.
A Go-driven run has no LLM to send, and its operator reads the Reason.

### D7 — The parent's fork decides

Decision: The brief instructs a one-shot fork to read the materials, then:
record each design call it makes with `lyx loom decision add` (D8);
record `lyx loom circling accept <slug>` or `lyx loom circling continue <slug>`;
resume the run (D9).
When it cannot settle the question, it records nothing and reports back to its parent session, which asks the operator; the run stays `awaiting`.
The `circling` verbs accept an escalation of either cause; the decision file records the cause beside the decision.
`accept` ends the segment unconverged on that round, exactly as the existing circling accept does.

Bound: `accept` lets the run proceed past one segment that may still hold open gating findings, including on a budget escalation whose verdict was plain `CONTINUE`; it passes only that segment, once, at the awaiting Bouncer row, and the committed decision file records it as unconverged with its cause, where the operator and later reviews can read it.
Shed's generic budget block stays the backstop for every other row.

Rationale: The operator's decision: a fork decides accept or continue and adds design calls; the operator is asked only when the parent cannot settle it or there is no parent.

### D8 — `lyx loom decision add`

Decision: A new verb `lyx loom decision add [<slug>] --by parent|operator --title <text> --decision <text> --rationale <text>` appends one entry to the decision record's `## Decisions` section:
an H3 heading `Added after Discussion (<by>, <UTC date>): <title>` over `Decision:` and `Rationale:` lines.
It then runs the same check as `lyx loom validate-discussion`; on a finding it restores the prior file content and refuses with the finding.
It commits the decision record through the discussion commit seam the Discussion-Review segment already uses.
The append is implemented in `internal/discussionparser`, the decision record's sole parser and writer.
The verb resolves the target worktree the way the `review` and `circling` groups do, and checks no caller identity, like them.

Refusals, each with a way forward and a `refusal-spec.md` row and reaching test:
no decision record exists;
the run's current producer is Discussion-Write and it is running (the writer owns the file; message it instead);
an empty title, decision or rationale;
a `--by` other than `parent` or `operator`.

Bound: append-only, one decision per call; it never edits or removes existing text, never routes the run, and never touches the support log or the plan.
It lets any shell addressing the run append to the answer key, the Plan-Write agent included, and `--by` is an unverified label, not an identity.
What is provided: every entry carries the `Added after Discussion` heading with its claimed label and date, and the append is a committed diff, so an addition is visible to a human reader in the record and in history.
Nothing checks the entries, and no reviewer is changed to read marked entries differently.
The entry lands in `## Decisions`, where Plan-Review already reads every Decision as answer key, so it licenses scope there; a Plan-Write addition is licensed by Plan-Review without a second reader, which stays an open risk.
A plan already approved is not re-planned by an addition; routing back is the caller's `lyx loom goto`.

Rationale: The `post-discussion-additions` note: an addition must enter the record, not the plan, so plan review reads it as answer key.
Routing through `discussionparser` keeps `PATTERN-sole-parsers`.

### D9 — Resume by the fork

Decision: After recording the decision, the fork resumes the run by running `lyx loom start` in the task worktree the brief names.
The circling verbs keep resuming nothing.
A batten-driven child is resumed the same way, which batten's documented halt handling already supports.

Rationale: `lyx loom start` is the existing resume, and its bootstrap is re-entrant, so no new resume path or batten change is needed.

### D10 — Who learns the new verbs

Decision: The Plan-Write stencil states that a scope addition arriving after Discussion is recorded with `lyx loom decision add` before planning on it, never written into the plan as an operator addition.
The escalation brief (D5) names the verb.
No other agent stencil names the `circling` verbs.

Rationale: The `agent-self-reflect` failure happened at Plan-Write; the brief is the parent's entry point.

### D11 — No GitHub issue for a recurring finding

Decision: The `recurring-finding` anomaly kind, its detector trigger, its ledger-observation input and its selfreport dedup branch are removed.
The other anomaly triggers stay, including `bounce-budget-exhausted`, which review segments no longer reach (D4) but `PR-Gate` still does.

Rationale: The operator's decision; issue #359 carried no state or history and duplicated what the escalation now delivers to the parent.

### D12 — Smoke tier

Decision: The failing smoke tests are fixed at their cause.
The working hypothesis is that the tests are stale: they count strands by exact name equality with `loom-status` and the driver role, while production names strands through `internal/agentname` and finds them with `agentname.Matches`.
If so, the tests match strands through `agentname.Matches`; if running them shows a production regression instead, production is fixed.
`TestSmokeStart_LLMRunRemovesEveryStatusStrand` is diagnosed the same way.

Rationale: The board entry asks for the tier green before further loom runs depend on it; the plan confirms the cause by running the tier, not by assumption.

## Constraints

- `PATTERN-review-round`: a segment converges only on a judge verdict; a parent `accept` ends the loop unconverged, the same as today's operator accept, and is recorded as such in the decision file.
- `PATTERN-sole-parsers`: the decision-record append lives in `internal/discussionparser`.
- `PATTERN-refusal-way-forward`: every new refusal of `lyx loom decision add` and every extended `circling` refusal names its way forward and lands with its `refusal-spec.md` row and reaching test.
- `PATTERN-stencil-ownership` and `PATTERN-shuttle-provider-seam`: brief and notice wording live in stencils read from the told stencils directory; `shuttleengine` and `reedengine` gain no provider words.
- `PATTERN-driver-choice-single-site`: neither the Bouncer nor Shed reads the recorded driver; `ly-drive` branches on the envelope field.
- `PATTERN-told-geometry`: the Bouncer is told the task worktree path and stencil paths; it derives none.
- `PATTERN-shed-producer-seam`: the `parent_notice` field is added in `shedengine` with stdlib only.
- `PATTERN-config-strictness`: the new `loom.yaml` keys go through `loomengine`'s existing strict `configengine.Load`.
- `PATTERN-cli-cobra`: the `decision` group mounts under `loom` with `Short` and JSON errors.
- `PATTERN-test-tier-purity`: guard, verb and config tests are untagged and spawn nothing; the smoke fixes stay in the `smoke` tier.
- `PATTERN-comment-line-breaks` and markdown semantic line breaks in every touched file.
- Docs land in the same commit as the code (CLAUDE.md).

## Auto-mode assumptions

- No operator answered; every choice below the board entry's own decisions is an auto-pick logged in the support log's question ledger.
- "Checkpoint" means every round from the checkpoint on, not exactly one round (D1).
- Both config keys go into `loom.yaml`, uniform across review segments (D3).
- A `continue` on a budget escalation buys one round, not a fresh budget (D4).
- The driver, not Go, delivers the notice (D6).
- The fork resumes with `lyx loom start` (D9).
- The smoke failures are presumed stale tests until running them says otherwise (D12).

## Open risks

- Config reach: the new `loom.yaml` keys land in this task's weft, and the `config-reaches-parent` note says weft config changes never reach the parent; the parent's `loom.yaml` may need `lyx config reconcile` before strict load accepts or defaults the new keys.
- `ly-drive` lives in `plugins/ly/skills/`, and the `board-labels-and-skills` entry moves it into a stencil; whichever lands second rebases the D6 edit onto the other's location.
- Under `discussion_interactive: false` the parent fork is an LLM; a fork that accepts too readily ends a review loop with open findings, visible only in the decision file and ledger.
- The D2 guard trusts the judge's own ledger keys; a judge that renames a key each round evades circling and reaches the budget escalation instead.
- A fork that records `continue` repeatedly extends a review loop one round per decision with no count cap; only the decision files show it.
- A `decision add` entry is unverified and no reviewer is changed to check marked entries; only the heading marker and the committed diff expose it.
- An unwired `Bounces` seam, or a read that is `ok` false or errored, at a spent budget falls to Shed's generic block, the one residual path that halts a review segment for a human.
- An awaiting run whose parent never answers stays awaiting without a bound, as `awaiting` does today.

## Acceptance criteria

- Below the checkpoint, the rendered judge prompt offers no `CIRCLING`; at and above it, it does.
- A `CIRCLING` verdict below the checkpoint is harvested as `CONTINUE` with a Warn.
- A `CIRCLING` verdict at the checkpoint whose ledger has no open key spanning the judged round and an earlier one is harvested as `CONTINUE`; one with a gating finding on such a key, unchanged or reopened (`rounds: [1, 3]`), escalates; a non-gating recurring key alone does not.
- A `CIRCLING` verdict at a spent budget escalates with cause `budget`, and a recorded `continue` on it is exempt for one round.
- At round N of a budget of N, a `CONTINUE` escalates with cause `budget` instead of blocking; a `continue` on a `circling` escalation below that round is counted and, with the seam returning `ok` true, never blocked.
- The `Bounces` seam's budget for a review row equals `review_max_bounces`, not the embedded recipe's or the engine default.
- With the `Bounces` seam unwired, `ok` false or erroring at a spent budget, a `CONTINUE` takes Shed's generic `bounce-budget-exhausted` block and does not escalate.
- A rising count over new keys at the checkpoint does not escalate.
- `loom.yaml` without the new keys resolves to checkpoint 3 and budget 5 through the template; a zero or negative value is a load error naming the key; review-segment rows take the config budget and `PR-Gate` keeps the recipe's.
- A `CONTINUE` at the spent budget returns `awaiting` with an escalation brief and `parent_notice`, not a `blocked` halt, and raises no `bounce-budget-exhausted` anomaly.
- A recorded `continue` on a budget escalation runs exactly one more round without blocking, and the next judge escalates again.
- A recorded `accept` on either cause returns Done after approve and commit.
- Re-calling the Bouncer at an undecided escalation rewrites nothing and returns the same `awaiting`.
- The status file and step envelope carry `parent_notice` on an escalation `awaiting` and omit it elsewhere.
- `lyx loom decision add` appends a valid entry, passes `validate-discussion`, commits the record, and leaves existing text byte-identical.
- Each `decision add` refusal is reached by a test and has a `refusal-spec.md` row.
- No `recurring-finding` anomaly is detected or filed for a ledger key open across three or more rounds.
- `go test -tags smoke ./internal/loomcli/` passes, and `go test ./...` passes.
- `ly-drive`'s SKILL.md describes relaying `parent_notice` to `LYX_PARENT`; the Plan-Write stencil names `lyx loom decision add`.

## Notes for the plan writer

- Bouncer verdict, circling decision and `Bounces` seam: `internal/shedadapters/bouncer.go`, `circling.go`, `bouncerfacts.go`, `bouncerprompt.go`; judge stencil `contracts/stencils/bouncer/bouncer-template-judge.md`.
- Circling verbs: `internal/loomcli/circling.go`; review-group worktree resolution: `internal/loomcli/review.go`.
- Shed's budget block and `Awaiting` persist: `internal/shedengine/run.go`; `BudgetExempt` already exists on the output pointer.
- Recipe rows: `contracts/recipes/loom-recipe.yaml` (embedded); assembly in `internal/loomrecipe`.
- Config: `internal/loomengine/config.go` and its template.
- Anomaly detector: `internal/loomengine/anomaly.go`; filing: `internal/loomcli/selfreport.go`.
- Smoke tests: `internal/loomcli/smoke_test.go` (`statusStrandCount` compares names by equality), `smoke_driverstrand_test.go`, `smoke_drivingsurface_test.go`; production lookup: `internal/loomcli/bootstrap.go` (`findStatusStrand`).
- Parent notification in `ly-drive`: `plugins/ly/skills/ly-drive/SKILL.md`, section "Notifying the parent".
