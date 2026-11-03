# Round 1 fixer-report — discussion

Every premise was checked against the code (`render.go`, `config.go`, `board.go:144-159`, `rename.go` header, `store.go` `MergeTasks`, `decision.go` arbiter and failed-commit paths, `wiring.go` `CarryOver`) and holds.

## Changes

- F1, fixed as proposed: D1 refuses every `design_prefix` that is a prefix of `decision-`, the empty one included, or that starts with `decision-`; the D1 criterion names an empty prefix, `dec` and a `decision-`-led prefix beside the equal one; Q11 in `support-log.md` matches.
- F2, fixed as proposed: D4's rename bullet moves the file after the save in the board's `afterSave` step, so a refused rename moves nothing; the D4 criterion adds that a refused rename leaves the file at the old name; the Notes line names D4's move beside its removals.
  D4 also states that a move that fails or is interrupted after the save leaves the new slug flagged without its file, and D2 names that case among the ways a flag without a file arises.
- F4, fixed differently: the Auto-mode line on older binaries states that their rename leaves the new slug flagged with no file, D10 refuses `--record` on a flagged entry whose file is missing, and the support-log D10 item is now fixed.
  Not made: the proposed way forward `lym loom decision store`. That verb stores onto the run's board slug, `shedrun.ResolveRunID`, which returns the worktree name (`internal/shedrun/resolve.go:20-24`) and does not follow a rename, so it meets D2's absent-slug refusal. The way forward names the old slug's `decision-<slug>.md` and the entry's next Discussion approval, which stores onto the new slug.
- F5, fixed as proposed: D4 gains a merge bullet keeping the `discussed` value with the file when a merge upserts a slug it removes; D3 judges such a payload against the removed entry's stored value; the D4 criterion names the kept flag.
- F6, fixed as proposed: D2's Bound names the approve seam (D6), `decision add` (D8) and `lym loom decision store` (D9), and points to D9's bound for the hand route.
- F7, fixed as proposed: D8 reports a copy refused on a done entry as `record_store_error` naming that a landed entry keeps its last stored record, with no pointer at `decision store`; D9 adds the done-entry refusal; the D8 and D9 criteria name the case.
- F8, fixed as proposed: D8's Bound says a goto past Discussion-Bouncer counts as past, so `decision add` copies; D6's Bound names D8 and D9; D9's Bound drops "only where the run's approved record exists". The D8 Rationale now notes the goto exception.
- F9, fixed as proposed: D8's carry-over sentence is replaced by the bound that the board copy is refreshed only at approval, on `decision add` and by `decision store`; the Goal says `decision add` refreshes the copy instead of keeping it current.
- F10, fixed as proposed, narrowed: the landed-despite-error path copies as a successful commit does, and the failed-commit refusal names `lym loom decision store` after `lym fabric commit` only on a run past its approval, since D9 refuses the verb on any other run; the D8 criterion names the failed-commit case.
- F11, fixed as proposed: D9 admits the arbiter by the arbiter-strand check alone, with no label, and the parent orch and the unnamed shell by `decision add`'s parent and unnamed-shell rules.
- `support-log.md` `## Review rounds` records round 1.

## Self-check

- fix-consistency, LOW, Review rounds still said no rounds while the Turn 4 item cites round 1: fixed as proposed.
- fix-consistency (aside), fix-proposal MEDIUM, fix-way-forward MEDIUM, merged: `lym loom decision store` cannot reach a renamed entry: fixed with own change, the D4, D10 and Auto-mode ways forward now name the old slug's file and the entry's next Discussion approval.
- fix-claims, MEDIUM, D2's "only from a rename" omitted a failed move and a hand deletion: fixed as proposed, D2 and D4 name a failed or interrupted move and a hand deletion.
- fix-proposal LOW, fix-way-forward MEDIUM, fix-claims LOW, merged: the failed-commit refusal named `decision store` unconditionally: fixed as proposed, limited to a run past its approval in D8 and its criterion.
- fix-proposal, LOW, the goto copy contradicted D8's Rationale: fixed as proposed, the Rationale names the goto exception.

## Disputed

None.

## Deferred

None.
