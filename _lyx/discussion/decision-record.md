# Decision record — review-loop-convergence

## Goal

Review segments stop circling to their budget on fresh edge-case keys.
A segment converges once its latest round carries no BLOCKING finding and no MEDIUM-or-worse `design` finding on a key already open in an earlier round, and the fresh findings it closes on reach the next producer through the decision record.
The template budget drops to three rounds, each review segment can set its own review and fix models, the fixer can be told to start only after the review is accepted, and a recorded circling decision is resumed by `lyx loom resume`, the step the hub's role already names (issue #446).

## Scope

In:

- The judge's decision rule (`decisionRuleMarker` in `internal/shedadapters/bouncerprompt.go`) and the judge stencil's convergence wording (`contracts/stencils/bouncer/bouncer-template-judge.md`).
- A Go-written carry-over of a closing round's open findings into the decision record's `## Open risks` section, on every Bouncer `Done` that settles a judged round.
- `review_max_bounces` template default from 5 to 3 in `internal/loomengine/template.yaml`, with its comment and the `review_circling_checkpoint` comment updated.
- Per-segment review and fix model keys in `loom.yaml`, falling back to `review` and `fix`.
- A `loom.yaml` option for when a round's fixer starts: beside the reviewer (default, today) or after the review is accepted.
- `lyx loom resume` over a run awaiting at a Bouncer row with a recorded circling decision; the circling verbs' `resume` output field, the escalation brief and the circling verb doc name `lyx loom resume`.
- PATTERN.md's `PATTERN-review-round` entry and `pattern/PATTERN-review-round.md`, plus the package `doc.go` files of every package whose observable behavior changes.

Out:

- Auto-accept at the round cap: dropped by the operator (relayed by `ly:orch`, 2026-10-08). A spent budget still escalates to the parent, unchanged.
- Lens fans for Discussion and Plan: task `review-lens-fan`.
- Per-segment judge models, per-segment budgets, and per-segment fixer start: nobody asked for them.
- `lyx loom approve` / `reject` outputs and PR-Gate's awaiting refusal in `loom resume`: unchanged.
- Batten's decision reading (`battenshed` `ReadDecision`): it keeps reading approvals and rejections only.

## Decisions

### D1 — Lighter convergence rule from round 2

Decision: from round 2 on, the judge rules `CONTINUE` while the latest round carries any BLOCKING finding, or a gating-class (`design`) finding at MEDIUM or worse on a key the facts list as open in an earlier round, unchanged or reopened; otherwise `CONVERGED`.
Round 1 keeps today's rule: `CONTINUE` while it carries any BLOCKING finding or any gating-class finding at MEDIUM or worse.
`decisionRuleMarker` renders the variant by round, as it already does for the CIRCLING bullet; the CIRCLING bullet and the checkpoint are unchanged.
The judge stencil's sentence "Fixed findings never count as convergence" is reworded to state the new bound: a fix converges only for a key first raised in the closing round, from round 2 on, and only with the carry-over of D2.
Bound: the rule lets a round-2-or-later round converge over fixes of fresh MEDIUM `design` keys no fresh reviewer has seen; it never lets through a BLOCKING finding, a recurring or reopened MEDIUM `design` key, or any round-1 fix.
Rationale: the 2026-10-08 escalations closed on APPROVED reviews whose only gating findings were fresh edge-case keys, so another round would only have raised more; keeping round 1 on today's rule gives every first-round fix, which carries the bulk of the changes, one fresh review.

### D2 — Carry-over of the closing round's open findings

Decision: when a Bouncer settles a judged round as `Done` (a `CONVERGED` verdict, or a recorded circling `accept`), Go appends one entry to the decision record's `## Open risks` section listing every `status: open` entry of that round's ledger at MEDIUM or worse: key, class, severity, rounds, and the round's review and fixer-report paths.
No entry is appended when the ledger has no such entry.
The append goes through `discussionparser` (a new write path beside `AppendDecision`, which writes `## Decisions` only), so `PATTERN-sole-parsers` holds.
It runs before Approve and Commit; a failed append returns settle's error, as a failed Approve does, so no segment passes with its carry-over lost.
The entry names its segment and round, and a re-call over a round whose entry already exists appends nothing.
It applies to all three review segments: the Discussion entry reaches Plan-Write, and the Plan-Review and Webster-Review entries reach Describe (the PR body the PR-Gate reviewer reads) and a rework generation.
The appended bytes land in a commit by the end of the Bouncer's `Done` transition in every segment, including Webster-Review, which has no commit seam today.
Rationale: the operator's precedent of 2026-10-08 recorded the remaining calls in the decision record by hand; Go writes it from the ledger the judge already produces, so no LLM output contract changes, and `## Open risks` is an existing section the discussion gate already accepts.

### D3 — Template budget of three rounds

Decision: `review_max_bounces` in `internal/loomengine/template.yaml` goes from 5 to 3; a spent budget escalates to the parent exactly as today.
Its comment and the `review_circling_checkpoint` comment state that with both at 3 a run reaches the budget escalation at round 3 before a CIRCLING verdict could change the outcome.
`review_circling_checkpoint` stays 3.
Test-kit defaults that mirror the template follow it where a test pins the template value.
Rationale: operator decision, 2026-10-08 ("it is mostly fine after three rounds"); the prime's own `loom.yaml` already carries 3.

### D4 — Review and fix models per segment

Decision: `loom.yaml` gains six keys, `discussion_review`, `discussion_fix`, `plan_review`, `plan_fix`, `webster_review`, `webster_fix`, each a `ModelSpecList` like `review` and `fix` (one model-spec, or a per-round list whose last entry serves every later round).
An empty value (`""`, the template value) takes the run-wide `review` or `fix` list; a set value replaces it for that segment's BurlerRound row.
Every set entry is validated through `modelspec.Parse` at load, as `review` and `fix` are.
The judge model stays the run-wide `judge` key.
Rationale: flat keys match `loom.yaml`'s existing shape and the strict loader's every-key-present gate; an empty value meaning "inherit" follows the `driver` and `friction` precedent.

### D5 — Fixer start option

Decision: `loom.yaml` gains `fix_start`, `parallel` (template default, today's behavior) or `after-review`.
With `after-review`, a round starts the fixer half only after the reviewer half has ended and Go has accepted the review and written the ready marker; the fixer's prompt is unchanged, and its first `lyx burler await-review` returns at once.
A reviewer half that ends without an accepted review starts no fixer, and the round reports the reviewer's outcome as it does today when the reviewer fails.
Any other value is refused at load, naming the two legal values.
The option is run-wide, applied to every review segment's rounds, and reaches `burlerengine` through `RunOpts`, so the engine never reads `loom.yaml` itself.
Bound: `after-review` only delays the fixer's start; it skips no gate, and the reviewer-before-fixer ordering of `PATTERN-review-round` is unchanged.
Rationale: the operator asked for the option only (2026-10-08); a run-wide key covers the cache-TTL case without per-segment surface nobody asked for.

### D6 — `loom resume` resumes a recorded circling decision

Decision: `lyx loom resume` over a run `awaiting` at a Bouncer row whose latest escalated round carries a recorded, not yet settled circling decision takes the same path as a halted run: it wakes the live, parked driver, whose step re-calls the Bouncer, which acts on the decision.
Its other refusals (no live driver, driver not parked, unfinished merge) apply unchanged.
Awaiting at a Bouncer row with no recorded decision refuses, naming `lyx loom circling accept` / `continue` and then `lyx loom resume`; awaiting at any other row keeps today's approve/reject refusal.
The circling verbs' `resume` output field becomes `lyx loom resume`, and the escalation brief's step 3, `internal/loomcli/circling.go`'s doc and every way-forward over an awaiting Bouncer row name `lyx loom resume` and the circling verbs, never approve or reject.
Bound: the new edge wakes a driver only for a Bouncer-row await whose decision file exists and is not settled; it gives no decision itself and never spawns a driver.
Rationale: issue #446: the verb named `lyx loom start` while the hub's role names `lyx loom resume`, and resume refused with PR-Gate verbs; a batten run leaves an escalated child with a parent notice to the driver's relay, so waking the parked driver is the step that moves the run and batten needs no change.

### D7 — PATTERN-review-round amended

Decision: `PATTERN-review-round`'s background bullet "a review segment converges only on a judge verdict over a fresh review, and fixed findings never converge on their own" is amended in the same commit as D1 and D2: a segment still converges only on a judge verdict, and from round 2 on a fix of a key first raised in the closing round may converge, carried to the decision record's `## Open risks`.
The PATTERN.md one-liner is reworded to match.
Rationale: D1 departs from the entry as written; PATTERN.md requires the change to be named and to land with the code.

## Constraints

- `PATTERN-review-round`: amended by D7; every other clause (review on disk before any target is touched, every finding fixed or disputed with evidence, reviewer and fixer separate sessions, the fixer touching nothing before the ready marker) holds, including under `fix_start: after-review`.
- `PATTERN-sole-parsers`: the decision-record append is a `discussionparser` function; ledgers and reviews are read by the code that reads them today.
- `PATTERN-config-strictness`: `loom.yaml`'s new keys load through the same strict path as its existing keys, and the template carries every key.
- `PATTERN-refusal-way-forward`: the new and changed `loom resume` refusals and the new config refusals name the way forward, with spec rows and a test reaching each refusal in loom.
- `PATTERN-no-denied-recovery`: no new way forward names a denied command.
- `PATTERN-shuttle-provider-seam`: the fixer start option lives in `burlerengine` and `loomengine`, never in `shuttleengine`.
- `PATTERN-stencil-ownership` and `PATTERN-producer-pointer-rule`: the judge stencil and escalation brief changes stay in their stencils; `decisionRuleMarker` stays the single holder of the rule's variants.
- `PATTERN-comment-line-breaks` and the repo's semantic-line-break rule apply to every touched comment and `.md` file.
- Docs land in the same commit: `doc.go` of `shedadapters`, `burlerengine`, `loomengine` and `loomcli` where their behavior changes, and `contracts/specs` rows for the changed refusals.

## Auto-mode assumptions

- Round 1 keeps today's rule (D1) although the board's proposal states the lighter rule without a round floor: applied from round 1, a first round would converge over its own unreviewed fixes.
- The carry-over covers every open ledger entry at MEDIUM or worse, of any class, not only fresh `design` keys, so a parent-accepted round carries its open findings the same way.
- The carry-over also runs on a recorded circling `accept`, so a parent accept and a convergence leave the same record.
- Webster-Review's carry-over goes to the decision record too, since Describe and rework already read it; no second sink is added.
- `fix_start` and the per-segment keys live in `loom.yaml`, not `burler.yaml`, since `burler.yaml` is a seed-only, operator-owned lens library and the models already live in `loom.yaml`.
- `review_circling_checkpoint` stays 3 after the budget drops to 3.

## Open risks

- From round 2 on, a fresh MEDIUM `design` fix that is wrong reaches the next producer unreviewed; the carry-over names it, and the Plan-Review rubric and the PR-Gate reviewer are where it is caught.
- With `review_circling_checkpoint` and `review_max_bounces` both 3, CIRCLING never changes a default run's outcome; an operator who wants an early circling escalation lowers the checkpoint.
- The judge maps findings to keys; a judge that gives a recurring finding a fresh key converges a round D1 means to continue. The facts' recurring-key list and the lossless ledger carry-forward rule are the existing guard.

## Acceptance criteria

- `decisionRuleMarker` for round 1 renders today's CONTINUE/CONVERGED rule; for round 2 and later it renders the lighter rule (BLOCKING, or MEDIUM-or-worse `design` on a key open in an earlier round); the CIRCLING bullet appears from the checkpoint on in both variants.
- The judge stencil carries no sentence that contradicts D1.
- A `CONVERGED` round whose ledger has open MEDIUM-or-worse entries appends one `## Open risks` entry naming segment, round, each key with class, severity and rounds, and the review and fixer-report paths; the decision record still passes `lyx loom validate-discussion`.
- A `CONVERGED` round with no such ledger entry leaves the decision record byte-identical.
- A recorded circling `accept` appends the same kind of entry.
- A replayed settle of the same round appends nothing a second time.
- A failed append returns an error and neither Approve nor Commit runs.
- The appended entry is committed by the end of the `Done` transition in Discussion-Review, Plan-Review and Webster-Review.
- The template's `review_max_bounces` is 3, and a CONTINUE at round 3 escalates with cause `budget`.
- Each of the six per-segment keys, when set, selects that segment's reviewer or fixer model per round; when empty, the segment uses `review` / `fix`; a malformed entry is refused at load naming the key.
- `fix_start: parallel` starts both halves as today; `fix_start: after-review` starts the fixer only after the review is accepted; a reviewer that fails under `after-review` starts no fixer; an unknown value is refused at load.
- `lyx loom resume` over a run awaiting at a Bouncer row with a pending circling decision and a parked live driver wakes the driver; with no decision it refuses naming the circling verbs and `lyx loom resume`; at PR-Gate it refuses as today.
- `lyx loom circling accept` / `continue` output `"resume": "lyx loom resume"`, and the escalation brief names `lyx loom resume`.
- `PATTERN.md` and `pattern/PATTERN-review-round.md` state the amended rule.

## Notes for the plan writer

- The Bouncer's settle path is `settle` → `settleUnconverged` / `settleUndecided` in `internal/shedadapters/bouncer.go`; the carry-over hooks in before Approve on both `Done` branches.
- Ledger entries with class and severity are already parsed for the judge facts (`bouncerfacts.go`); the round's review and fixer-report paths are known to the run directory helpers in `bouncerfiles.go`.
- `discussionparser.AppendDecision` in `internal/discussionparser/append.go` is the shape to follow for the `## Open risks` append.
- Model lists flow `loomengine.Config` → `shedrecipe.Env.ReviewModels` (`burlerengine.RoundModels`) → the BurlerRound row; the per-segment lists need the row's segment at that point.
- The fixer's start is in `burlerengine` `Engine` run (`roundHalves`, then two `StartGated` calls, then `join`).
- `loom resume`'s awaiting refusal is in `internal/loomcli/resume.go`; the circling decision reader is `readCirclingDecision` in `internal/shedadapters/circling.go`, and `loomrecipe.BouncerRunSubdir` resolves a row's run directory.
