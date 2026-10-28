# Decision record — landing-settles-in-place

## Goal

A landing settles what it can in place instead of rerouting or handing the stop straight to the orch:
Finalize merges a moved parent itself, conflict and overlap included, with no reroute, and a landing merge fixer's `escalate` goes to the arbiter first, so only the arbiter's hand-on reaches the orch.

## Scope

In:

- `internal/landingshed` Finalize: a conflict is resolved by the conflict session under the parent lock, and a large merge (conflict or overlapping files) goes on to the `finalize` gate, for every recipe.
- Removal of the per-recipe large-merge flags and the dead paths they leave: `Deps.ConflictReroutes`, `Deps.LargeMergeLands`, Finalize's abort-on-conflict merger seam, the overlap note and darn's overlap-note review input.
- A distinct reason prefix for a landing fixer's `escalate` halt at Parent-Align and Finalize, read by `landingshed.HandsBack` as no hand-back, by the arbiter seam as judged first and by batten's binary re-step as left alone.
- `sheddriver.ArbiterSeam.JudgesFirst` takes the stop's reason as well as its producer; loom's wiring (shared by loom, darn and pawl) marks Webster as today plus the landing escalate.
- Docs, per D5.

Out:

- The fixer's spent cap, a failed or unusable fixer turn, an over-cap change and an undecodable attempt record: their routing, halts and hand-backs stay as they are, the Finalize `on_reroute` edges of loom and darn included.
- Parent-Align's own merge behaviour, its place before Code-Review and its fixer.
- The reroute-cap halt at Finalize, which stays a hand-back.

## Decisions

### D1 — Finalize merges a moved parent in place for every recipe

Finalize always merges the parent through its resolver: a conflict starts the conflict session under the parent lock, as darn and pawl do today.
A merge that needed the conflict session, or whose parent changes touch files the task changed, is no longer a reroute cause for any recipe: Finalize writes the merge record (overlap included, as now) and goes on to the `finalize` gate, whose failure runs the merge fixer in finalize mode.
The retry after a merge-in-required error from the parent-side merge follows the same path: it merges, gates and lands, and no longer reroutes on a code-side commit.

Rationale: the operator's design (board entry, 2026-10-25): Parent-Align before Code-Review takes the bulk of the parent's drift, and Finalize takes only what moved since, without re-running the Code-Review perch.

Bound (a guard removed): what now lands with no review is the conflict session's resolution against the parent's newer work and a textually clean merge of parent changes to files the task also changed, in loom and darn as already in pawl.
The `finalize` gate over the merged tree bounds both; a failing gate runs the fixer, whose attempts and size and kind caps bound its unreviewed change, and a spent cap or over-cap change still reroutes to a review or halts for the orch.
The conflict session runs while the parent lock is held, so sibling landings onto that parent wait for it, which can lengthen a landing's wall time; the lock's existing owner, pause and release rules bound that wait unchanged.

### D2 — The per-recipe large-merge flags and the overlap note are removed

`Deps.ConflictReroutes` and `Deps.LargeMergeLands` are removed, with `landingDeps`'s settings of them, and Finalize keeps one merge path.
Finalize's abort-on-conflict merge seam (`taskMerger` and its field) is removed, since no recipe aborts a conflict any more.
The overlap note (`WriteOverlapNote`, `OverlapNotePath`, `overlapnote.go`) is removed, with `overlapNoteInput` and darn's override of `ReviewInput`; darn then keeps the shared wiring's Parent-Align fixer-report seam, which answers not found on a darn run since it has no Parent-Align.

Rationale: once D1 holds for all three recipes both flags are constant and the note has no writer; keeping either would be a seam with one permanent value.

### D3 — A landing fixer's escalate halt carries its own reason prefix and is not a hand-back

A fixer `escalate` at Parent-Align or Finalize halts with a reason opening with a new exported prefix in `landingshed`, distinct from `LandingStopPrefix`; its text still names the row, the report path and the conflict, and its way forward reads: record the deciding decision with `lym loom decision add` in the task worktree, then `lym loom resume`.
`landingshed` exports one predicate that is true for a blocked stop at Parent-Align, Finalize or the retired Merge-In name whose reason opens with that prefix; it is the only reader of the prefix.
`landingshed.HandsBack` does not match the prefix, so the loop's envelope gets no `hand_back` mark, the driver's `HandsBack` rule row does not fire, and `arbiterstop` reads it as no landing halt, eligible as a blocked stop.
Batten's binary re-step (its `HandBack` seam in the batten CLI wiring) skips a stop the escalate predicate matches as it skips a landing halt, so a new build never re-runs the row over an undecided collision; only the arbiter's resume or the orch's `lym loom resume` moves it.

Rationale: the board entry's design: the landing-halt rule row "must yield for that case", and `HandsBack` is the one predicate the envelope mark, the driver and `arbiterstop` read, so one change keeps them agreeing; batten's re-step reads the same predicate for a different purpose, leaving a stop to the parent, and an undecided collision still needs that.

Bound (an edge added): only a fixer's `escalate` report changes route; every other halt at those rows keeps `LandingStopPrefix` and its hand-back.

### D4 — The arbiter judges a landing escalate first, through the existing seam

`sheddriver.ArbiterSeam.JudgesFirst` becomes `func(producer, reason string) bool`, the reason being the stop's reason, which the driver's stop already carries.
`loomArbiterSeam` marks a blocked stop at Webster, as now, or one D3's predicate matches.
Such a stop goes rule table, arbiter, then parent, with no tackler call; a blocked stop at Parent-Align or Finalize with any other reason still meets the tackler first.
The arbiter settles it with its existing repair verb `lym loom decision add` and a `resume` whose goto is the halted row; the re-run fixer reads the decision record afresh, its escalate turn already given back.
A hand-on, an arbiter that is off or out of spawns, and every arbiter fault hand the stop back to the orch through the driver's existing hand-on path.
A pawl run has no decision record to add to, so its arbiter hands an escalate on.

Rationale: the board entry: the arbiter can likely settle a decision collision against the decision record and what landed, `JudgesFirst` is the seam for "arbiter first", and a landing escalate "is one more row for the seam, not a new mechanism". The reason is needed because a row-only mark would send every blocked Finalize stop past the tackler.

Bound (an edge added): the arbiter can now let a landing proceed past a decision collision with no orch turn.
It is bounded by the arbiter's own rules (it hands on any call needing an operator decision the board entry and decision record do not record, or that widens scope), by its per-run spawn cap, which stops an escalate-resume loop, and by the re-run fixer's gate and caps.
The driver's build hand-over row precedes the arbiter-first row, as for Webster's stop today: while the driver lives, a new build re-steps the escalate stop once before the arbiter sees it, and a re-escalation then reaches the arbiter under its new stop key.

### D5 — Docs and role text follow the behaviour

- The orch role stencil: `Landing halts` says a moved parent, conflict or large merge no longer reroutes, and a fixer's escalate reaches the orch only as the arbiter's hand-on; its landing-stop bullet drops "escalated a decision"; the parent-lock bullet's "reroutes if the parent moved meanwhile" says the released run merges the moved parent at its next Finalize.
- The loom and darn recipe header and Finalize row comments: the `on_reroute` edge carries only a spent or over-cap fixer change.
- The Code-Review rubric's review-input paragraph drops the overlap note.
- The landing fixer report spec: an `escalate` goes to the arbiter first.
- `docs/overview.md`'s Finalize and landing-halt lines.
- `internal/landingshed/doc.go`, `internal/arbiterstop/doc.go`, `internal/sheddriver/doc.go` (the JudgesFirst rule row and the loom-wiring sentence) and `internal/battenshed/doc.go`'s binary re-step paragraph.
- The `PATTERN-shed-stop-chain` entry, per PATTERN changes.

Rationale: CLAUDE.md: docs land in the same commit as an observable behaviour change.

## Constraints

- `PATTERN-shed-stop-chain`: its arbiter-first clause is per row; D4 makes it per row and reason, so the entry changes (see PATTERN changes).
- `PATTERN-parent-lock`: merging into a local parent only in Finalize under the parent lock; D1's conflict session runs inside that lock, which complies.
- `PATTERN-refusal-way-forward`: the new escalate halt text keeps a way forward an agent's settings allow.
- A run already halted with an escalate reason under `LandingStopPrefix` from an older build stays a hand-back to the orch, which is today's behaviour; no migration of halted reasons.
- A darn run an older build rerouted on a large merge loses its overlap note as review input once upgraded; its re-entered perch reviews the diff alone, which is accepted.
- Pawl's landing keeps its own treatment: it already resolves under the lock and lands a large merge through the gate, and its nil `RerouteReviewed` still halts a spent or over-cap fixer.

## Auto-mode assumptions

- The removed flags, seam and note are read only where Scope and D2 name.
- The arbiter stencil needs no change: its "a goto at or before the current row" admits the halted row itself.
- Finalize keeps writing the merge record's `Overlap`: the fixer stencil reads the merge record file, overlap included.

## Open risks

None.

## Acceptance criteria

- A loom Finalize whose parent moved since Parent-Align with changes to files the task changed, over a passing `finalize` gate, lands with Done and no reroute.
- A loom Finalize whose merge of the parent conflicts resolves the conflict with the conflict session under the parent lock, gates the merged tree and lands, with no reroute and no abort.
- A darn Finalize in either case lands the same way and writes no overlap note.
- A large merge whose gate fails runs the finalize-mode fixer; a fixed change within the caps lands, and a spent cap or over-cap change still reroutes when a review follows and halts with `LandingStopPrefix` when none does.
- The retry after a merge-in-required error merges, gates and lands a code-side parent commit instead of rerouting.
- A fixer `escalate` at Finalize and at Parent-Align halts with the escalate prefix; `landingshed.HandsBack` is false for it, the envelope is not marked `hand_back`, and `arbiterstop.Check` admits it.
- The driver sends such a stop to the arbiter with no tackler call; an arbiter `resume` with the row as goto re-runs it, and a hand-on, an arbiter off and a spent arbiter budget each hand it back to the orch.
- Batten's binary re-step leaves a halted escalate stop alone on a new build.
- Any other blocked stop at Parent-Align or Finalize still meets the tackler first, and Webster's blocked stop still goes to the arbiter first.
- Every other landing halt (spent, over-cap, unusable turn, undecodable record, lock re-check) keeps `LandingStopPrefix` and its hand-back.

## PATTERN changes

- `PATTERN-shed-stop-chain` — changes: A shed run's stop: rule table, tackler, arbiter, then parent, for every recipe; a blocked stop the driver's arbiter seam marks as judged first, by its row and reason, goes rule table, arbiter, then parent; arbiter keys in shed.yaml with module overrides.

## Notes for the plan writer

- Finalize's large-merge `switch` in `mergeParent` and the `ConflictReroutes` branch of `mergeIntoTask` are the code D1 removes; `fixerOutcome.haltReason` is where D3's prefix goes.
- `landingDeps`'s drift-guard test pins every `Deps` field it fills, so it changes with D2.
- The batten CLI wires `arbiterstop.HandsBack` as batten's child `HandBack` seam; D3's re-step skip goes there.
