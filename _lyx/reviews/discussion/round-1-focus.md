---
round: 1
focus:
  - "D2 and D6/D8: a requested cycle skips the re-read and the failure hold. Settle whether each Decision states what that path can now let through (a cycle on a fresh, small reading) and what bounds it."
  - "D11 and D6: the claim that background tasks survive `/compact` is a premise of dropping watcher re-arming and of D7's dropped DEFER. Settle whether it is stated as an Auto-mode assumption or Open risk, or only cited from the `/clear` smoke run."
  - "D12 and D13: notice queue lifecycle. Settle whether the Decisions bound a stale or dead orch strand recorded in orch state (notices queued with no deliverer), episode state across a batten restart, and what a notice can make the orch do through its typed text."
  - "D13 and D16: delivery is gated on the idle probe. Settle whether a permanently failing probe (too-short pane) holding notices and cycles indefinitely is bounded or surfaced, given the new PATTERN invariant makes the watcher the only writer."
  - "D10 and D5: a config load error on a stale `permission_mode: prompt` or bad `cycle_mode`. Settle whether the failure mode for an already-running hub (restart on an old `orch.yaml`) is stated."
  - "D15: fail-closed on an unreadable start time versus the Auto-mode assumption that the registry records a process start. Settle whether the decision still holds if that assumption is false."
  - "Relocation pass over `decision-record.md`: check Rationale lines, Open risks and Notes for deliberation narrative or incident history that is not a requirement or constraint, applying the completeness-before-leanness test before suggesting any move."
  - "Consistency pass: Scope bullets against Decisions D1-D18 and Acceptance criteria (every Scope item has a Decision and a criterion; no criterion tests behavior no Decision states)."
---

Seeds only; the reviewer judges every item against the full rubric.
None of these concerns a missing "Notes for the plan writer" subsection, a rejected alternative absent from the record, or call-site enumeration, and none asks for signatures or `file:line` citations the discussion stencil told the writer not to gather.
