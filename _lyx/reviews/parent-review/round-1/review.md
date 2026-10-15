---
verdict: APPROVED
---

## Scope against the board entry

The design covers every point of `refusal-route-config-empty`, and nothing beyond it.

- Part 1, label: D1 replaces `plan_drifted` at begin-batch with `plan_conflict` and a cause, `plan_defect` or `tree_moved`, which separates a plan wrong at authoring from a tree the run moved.
  record-batch and recover-batch keep `plan_drifted` for real fingerprint drift.
- Part 1, route: D2 and D3 route a blocking dispatch finding under loom to `Plan-Burler-Review` through the Webster row's `on_reroute`, instead of halting for a human.
  The way forward the refusal names stays unchanged.
- Part 1, earlier check: the entry only asks to consider it.
  The design declines it with a stated reason: the plan gates already run delete-before-reference.
  That is a considered answer, not a left-out point.
- Part 2: D6 refuses every `lyx config` editor path without a terminal and names `--print` and `--set`.
  D7 never saves an empty or whitespace-only editor result over a layer.
  Guarding `lyx config menu` as well stays inside the entry, since menu reaches the same editor path.

No other board entry owns this work.
`delete-order-own-edit`, the false-positive fix the entry cites, has already landed, and this design does not redo it.
`run-verbs-shared`, `orch-split`, `weaver`, `loom-model-key-names` and `role-language-skills` touch loom, the driver or config, but none of them touches begin-batch's refusal, Webster's routing or the `lyx config` editor path.

## Overlap with runs in flight

`lyx loom peers` lists one peer and no parent commits.

- `run-pane-timing` (row 3, own row 3: a tie, so the landing order is unknown and whichever lands second takes the other's changes at Merge-In).
  Shared files:
  - `contracts/recipes/loom-recipe.yaml`: the peer adds `role_prefix` to every review row, `Plan-Burler-Review` included, and this run adds `on_reroute` to the Webster row and edits `Plan-Burler-Review`'s instructions.
  - `internal/shedadapters/webster.go`: the peer copies the verify gate's count and spans, and this run adds the reroute arm.
  These are different keys and different arms of the same files, so the overlap is text only.
  The peer's D7 rework attribution keys on a Stuck entry with `routed_to`, while a reroute writes an `OutcomeReroute` entry, so the two designs do not change the same behaviour.
  This run's decision record already names the pane-rendering interaction under Open risks.
  No decision in either record contradicts or duplicates one in the other, and neither changes the same PATTERN entry.
