---
round: 1
focus:
  - "D2 gate verdict table: check whether the row precedence is fully stated (merged PR, matching approval, matching rejection, stale records of either kind, closed PR, malformed record) and whether every combination the operator can produce by running approve and reject in sequence has exactly one verdict."
  - "D3/D4 bounce budget: the gate never returns Done between rejection rounds, so its episode Stuck count accumulates across every round. Settle whether D3, D4's exhausted-budget refusal, and Routing.Bounces (which counts the segment's main-line row) agree on what 'five' limits and on the state an exhausted run is left in."
  - "D3/D5 rework row failure path: the rework row has no on_stuck, and its append-only check returns Stuck. Settle whether the decision record says how an operator recovers a run blocked at the rework row, given that reject refuses anywhere but the gate row and approve checks only the gate."
  - "D5 plus Open risks: the crash-between-commit-and-record-deletion re-entry requirement appears in Open risks and Acceptance criteria but not as a Decision. Check whether the plan writer gets a decided mechanism for detecting that the pending rejection's round is already committed, or only a stated risk."
  - "D8 batten decision identity: check whether the record defines the identity a rejection is resumed once per, and whether that identity stays distinct from an approval at the same PR number and head (for example, approve after reject at an unchanged head)."
  - "Auto-mode assumptions, last bullet: a plain loom start no longer pushing a hand-made fix is an observable behavior change. Settle whether it is an assumption or a decision with docs consequences (ly-drive, overview), and whether it belongs in Decisions or Constraints."
  - "Acceptance criteria versus D4: compare the reject refusal list in Acceptance criteria against D4's refusal table (for example, 'origin URL unusable') and settle whether each D4 refusal has a matching criterion."
  - "Decisions rationale lines (for example, D2's 'no engine change' and D6's '--fresh would discard every completed batch'): settle whether any Rationale restates a rejected alternative that belongs in support-log.md's Rejected alternatives, after first checking it carries no constraint the plan writer needs."
---

Seed focus for round 1, drawn from a first read of both files against the Discussion-Review rubric and the writer stencil.

The design leans on shedengine's episode-based bounce counting (`episodeStuckCount` resets only on a Done by the same row) and on the OnStuck same-segment rule in `internal/shedengine/validate.go`.
Several entries above ask whether the record's routing and budget statements hold together under that counting.
The questions are about completeness and internal consistency of what the plan writer receives.
None of them ask for signatures, file:line citations or call-site enumeration, which the writer stencil excludes.

Checked against the Do-not-flag list: the Notes for the plan writer subsection is present and not a subject here, rejected alternatives are already in `support-log.md`, and no entry asks for call-site enumeration.
Skipping the Batchifier row on the rework path was checked and dropped as a focus: Webster resolves the active batcher itself in `internal/loomshed/webster.go`, so the Batchifier row is only a config-resolvability gate.
