---
round: 3
focus:
  - "D7 (`_lyx/discussion/decision-record.md`): does the move hold the parent lock of the worktree's branch and the weft write lock from the move record's write through the last step, so that a child loom run's Finalize (which takes the parent lock and commits parent run records at the parent's working-tree anchor) cannot write into the weft mid-move? Settle whether the acceptance criterion states holding, not waiting."
  - "D7 steps 1 and 2 with the prime refusals: does the wait cover every run of every dobby that step 2 ends, and does it read a pass as in flight from the process table as well as the driver lock, so step 2 cannot kill a pass spawned before the record that does not yet hold its driver lock?"
  - "D7 recovery, Not-landed case: does the recovery restore the machine-local `.lyx` (a wired, never-tracked name) before any reset or deletion under the moved names, and is `.lyx` excluded from what the reset and the untracked deletion act on?"
  - "D7 recovery landed test for an integration: is the tip it compares against recorded after the wait and any weft pull or commit that can intervene, so an unlanded integration cannot read as landed?"
  - "D7 and D5, bounds on other starters while a move record stands: do `lyx adopt` (Topology.Add, first passes, `lyx loom resume`, the finalize gate) and `Add` (`lyx fabric add`, `lyx beam create`) refuse before touching anything, with a named way forward, and does the acceptance criteria list each refusal?"
  - "D10 and D5, markerless tip: does every place that compares or reads a marker (pull and push stop, the integration's source anchor, D7's `A` for a markerless worktree) use one defined value for a markerless tip?"
  - "D10 way forwards: does the checkout refusal's way forward avoid re-anchoring the branch being left, and does the integration conflict's way forward preserve every local commit after the conflicting one?"
---

The focus lists where round 2's findings clustered, as questions to settle against the current text.
