# Alignment agent settles landing order itself; escalates only on design conflict

`alignment-landing-order` · Note · enhancement, orch, loom

The alignment agent escalates to the orch on cross-run overlap with "which run lands first?".
The orch answers by one rule nearly every time: the run furthest along lands first, and the other takes its changes at Merge-In.
On 2026-10-12 it woke the orch twice for such overlaps (`cleanup-1012` and `convergence-plan` against other runs in flight), and both were settled by that rule with no design call.
The facts the rule needs are already readable to the agent: the board, `lyx loom peers` and each peer's decision record.

## Idea

The alignment agent applies the landing-order rule itself and records the outcome in its report.
It escalates to the orch only when the designs conflict: both runs change the same behavior in different ways, or a recorded decision in one contradicts the other, so a parent decision is needed in the later run.
A text-only overlap (same file, different lines or an import and a mount on each side) is never a reason to escalate.

The operator agreed the orch should get only stops that need a parent choice (2026-10-12); see the note `arbiter-when` for the matching rule on arbiter spawns.
