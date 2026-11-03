# Fixer report — discussion round 1

F4 (`decision`) went to the parent and is left out here.

## Changes

- F1, fixed as proposed (D1 option: a block row's `on_done` may be empty).
  D1 drops `on_done` from Parse's required keys and lets a block row's `on_done` be empty inside another block, refused empty outside every block.
  D2 states innermost-first expansion, defines a block's exit as its block row's `on_done` or, when that is empty, the enclosing block's exit, and rewrites every empty inlined `on_done` and inlined block row's empty skip target to the exit of the innermost enclosing block.
  The edge refusal reads each row's edges as its own recipe file sets them, exempting the edges expansion sets.
  The D1 and D2 acceptance bullets carry the change, including the nested-block row.
- F2, fixed as proposed.
  The Goal and D1 say the block ends when the run reaches its exit.
  D3 says the block row's Done marks entering the block, and `lym loom status` shows it as entered.
  D4 is renamed "A block's end and Stuck". It widens the end condition to a Done or a skip reaching the exit, and says a reader finds the end as the exit's first entry after the block row's latest Done entry.
  The D3 acceptance bullet and a new D4 bullet match.
- F3, fixed differently in one point.
  The proposal reads "only `lym loom goto` bypasses the block gate". A driver's re-launch onto an inlined row also runs that row without the block gate held (`internal/shedengine/goto.go:241-251` moves the run, and the gate it re-evaluates is the target's own).
  So the Bound names both moves: only a goto or a driver's re-launch bypasses the gate, both move only a halted run to a row `CheckGotoTarget` admits, a move into a passed-over block is the mover's decision, and a re-launch onto the block row evaluates the gate again.
  The proposal's "only back to a row" became "only to a row", because `gotoAdmitted` also admits the routed row (`goto.go:339`).
  D3's goto sentence and the acceptance bullet carry the re-launch case.
- F5, fixed as proposed: D3 adds the reconstructing-reader sentence, and the plan-writer note names the goto admission in `internal/shedengine/goto.go`.
- F6, fixed as proposed: D2's refusal list and its acceptance bullet refuse a block row whose `on_done` names itself or a row its own expansion inlines.
- F7, fixed differently in one point.
  The proposal reads "only a top-level key of an embedded recipe sets it". D8 lets any recipe set the key and names no embedded-only rule, so the Bound says "only a recipe's top-level key sets it".
  The rest is as proposed.
- The support log's Review rounds section records round 1.

## Self-check

- fix-proposal, MEDIUM: D4's end condition missed a nested block row passed over to the exit. Fixed as proposed: D4 and its acceptance bullet add "an inlined block row whose skip target is the exit is passed over".
- fix-proposal, MEDIUM, merged with fix-claims, MEDIUM, and fix-way-forward's out-of-lens note: D4's reader rule, "the first entry whose block path does not include the block", read a goto out of the block, or a pass-over, as the block's end. Fixed with my own change: the reader takes the exit's first entry after the block row's latest Done entry, and the block-path clause is dropped.
- fix-consistency: no findings.
- fix-way-forward: no findings in its own lens. Its out-of-lens note is merged above.

## Disputed

None.

## Deferred

None.
