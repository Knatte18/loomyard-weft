# CIRCLING escalates to the parent orchestrator, later and on recurrence only

`circling-to-parent` · Notes · feature

Seen 2026-10-04 in `board-model`: Discussion-Bouncer judged round 2 CIRCLING because gating findings rose from 1 to 2, both in the D10 intake bound that round 1's fix had widened; the run halted for a human.
The hub settled it with five short design calls and the run went on; no operator input was needed.
Operator (2026-10-04): halting on that is disproportionate.
Idea: CIRCLING is judged no earlier than round 3 and only when the same finding keys recur unchanged, not on a rising count; and a CIRCLING verdict escalates to the parent orchestrator like a parent-review request (message, fork decides accept/continue with design calls), reaching the operator only when the parent cannot settle it.
Related: `post-discussion-additions` (a verb for the parent's decisions to enter the decision record, instead of the driver editing it by hand).
