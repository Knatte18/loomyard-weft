# Support log — landing-settles-in-place

## Interview

Autonomous run: no operator turns; every question below was settled by the writer and logged in the question ledger.

### Turn 1 — scope from the board entry

Read the board entry (two former notes with operator designs dated 2026-10-24 and 2026-10-25); no `Decision for landing-settles-in-place:` note exists on the board.
Explored `internal/landingshed` (Finalize merge, gate and fixer paths, `HandsBack`, overlap note), `internal/loomcli/landingdeps.go`, `internal/arbiterstop`, `internal/sheddriver`'s rule table and `JudgesFirst`, `internal/loomcli/drivewiring.go`, the arbiter stencil and repair verbs, the recipes and the orch role stencil.
`lym loom peers` lists no peer touching these paths and no parent commits.

### Turn 2 — design questions batch (auto-picked)

- Q1: keep the per-recipe flags with new values, or remove them? Recommended and picked: remove (D2).
- Q2: keep Finalize's `on_reroute` edges? Picked: keep; the spent and over-cap fixer path still uses them and is out of scope.
- Q3: how does the escalate halt yield the hand-back row? Picked: a distinct reason prefix that `HandsBack` does not match (D3).
- Q4: how does the arbiter seam tell the escalate from other Finalize stops? Picked: `JudgesFirst` takes the reason (D4).
- Q5: does the arbiter need new verbs or stencil text? Picked: no; `lym loom decision add` is already a repair verb and a resume goto at the halted row is admitted.
- Q6: migrate older halted escalate reasons? Picked: no; they stay hand-backs as today.

### Turn 3 — lens review

All seven forks returned a report.

- existing-paths 1 and bound-breaker 1 (MEDIUM, merged): batten's binary re-step reads `HandsBack` and would re-run an escalate after the arbiter's hand-on — fixed: D3 makes batten's re-step skip the escalate (its seam is used by the re-step alone), and D4 states the driver's build hand-over as an accepted bound, as for Webster.
- existing-paths 2 (LOW): name batten's re-step as a reader — fixed with the above, in Scope, D3 and Notes.
- interactions 1 and walkthrough 1 (MEDIUM, merged): the orch role's parent-lock bullet still says a released run reroutes — fixed in D5.
- interactions 2 (LOW): the escalate way forward named no command — fixed: it names `lym loom resume`.
- interactions 3, walkthrough 2 (LOW, merged): darn's `ReviewInput` after the override goes — fixed: D2 says darn keeps the shared Parent-Align fixer-report seam, which answers not found.
- walkthrough 3 (LOW): pawl's arbiter has no decision record — fixed: D4 says it hands on.
- bound-breaker 2 (LOW): wrong reason for keeping `Overlap` — fixed in Auto-mode assumptions.
- bound-breaker 3 (LOW): in-flight darn runs lose the overlap note — fixed: accepted under Constraints.
- goal-scope 1 and leanness Open risks (merged): both risks restated D1's bound — fixed: the wall-time clause folded into D1's bound, Open risks now `None.`
- leanness Scope/D5 duplicate docs list — fixed: Scope says "Docs, per D5".
- leanness Constraints restating untouched entries — fixed: cut.
- leanness D1 rationale narrative — fixed: trimmed to the design citation.
- leanness arbiter stencil said three times — fixed: kept in Auto-mode assumptions only.
- leanness Notes repeat of `taskMerger` — fixed: removed.
- leanness Auto-mode assumption 1 snapshot date — fixed: reworded.
- leanness support-log Turn 2 duplicates the ledger — rejected: the interview turn records the batch as asked; the support log is not plan input.
- board-fit: no findings.

## Rejected alternatives

- Set `LargeMergeLands` true and `ConflictReroutes` false for loom and darn, keeping the flags: both become constant across every recipe, a seam with one permanent value.
- Keep the overlap note for darn as an informational file: no reader once darn no longer reroutes on a large merge.
- Remove Finalize's `on_reroute` edges: the spent and over-cap fixer path still reroutes to a review, which the board entry keeps.
- Keep `JudgesFirst(producer)` and mark Parent-Align and Finalize by row: every blocked stop at those rows, a dirty tree or a failed sync included, would skip the tackler.
- A new arbiter-first mechanism beside `JudgesFirst`: the board entry asks for one more row for the existing seam.
- Keep the escalate under `LandingStopPrefix` and special-case it inside `HandsBack` by its text: the escalate cause text is prose, and a prefix is the established way these predicates read reasons.
- Rewrite older `LandingStopPrefix` escalate halts on resume: one-off compatibility for a pre-release, single-operator system; the orch handles them as today.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick): remove `ConflictReroutes`, `LargeMergeLands`, the `taskMerger` seam and the overlap note with its darn review input — resolved, D2.
- Q2 (auto-pick): Finalize's `on_reroute` edges stay for the spent and over-cap fixer path — resolved, Scope.
- Q3 (auto-pick): a new escalate reason prefix that `HandsBack` does not match — resolved, D3.
- Q4 (auto-pick): `JudgesFirst` gains the stop's reason — resolved, D4.
- Q5 (auto-pick): no arbiter stencil or repair-verb change — resolved, D4 and Auto-mode assumptions.
- Q6 (auto-pick): no migration of older escalate halts — resolved, Constraints.
- Q7 (auto-pick): Finalize keeps writing the merge record's `Overlap` — resolved, Auto-mode assumptions.
- Q8 (auto-pick): batten's binary re-step skips a landing escalate, while the driver's build hand-over re-step stays as for Webster — resolved, D3 and D4.
- Q9 (auto-pick): darn keeps the shared fixer-report review-input seam rather than a nil one — resolved, D2.
