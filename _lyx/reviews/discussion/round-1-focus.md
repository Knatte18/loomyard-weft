---
round: 1
focus:
  - "D4 vs Auto-mode assumptions: D4 records both vcs.revision and vcs.modified and sets binary_changed when the identities differ, while the dev-build assumption says a modified tree shares its base revision and so triggers no re-step. Settle whether the modified flag is part of the compared identity, and whether decision-record.md states one rule the plan writer can implement."
  - "D5 clean-tree guard vs D6/D9/D11: the driver's own hand-back writes (stop report, repair records, commit-records) land around the moment the guard runs `git status --porcelain`. Settle whether the driver's own artifacts can leave the worktree dirty and permanently suppress the binary-change re-step, and whether the record says which paths the guard counts."
  - "D1's 'never text-matched' title vs the Open risks entry that lyx classifies git and GitHub failures by error text. Settle whether the decision record is internally consistent about where text matching is and is not allowed, so the plan writer does not read D1 as forbidding the D2 classification it needs."
  - "D3, D5, D8 and D10 together define the parked driver's state machine (immediate transient re-step, stop-time and watch-time binary re-step, resume line resetting budgets, watch re-arm on timeout). Settle whether every transition and budget reset is stated once and without contradiction, e.g. what 'behave as today' means in D8's live-driver-without-marker branch, which the matching acceptance criterion reads as 'neither sends nor spawns'."
  - "Constraints section: settle whether each named CONSTRAINTS.md invariant exists under that name and says what the entry claims (notably the Shed Producer-Seam import list, the Shed Verb-Set closed key sets, and the Driver Choice Single-Site Invariant)."
  - "Acceptance criteria vs D3, D7, D10 and D11: several behaviours live only in the skill (driver removes the marker before a self re-step, watch re-arm, one re-step per transient stop, repair-record contents). Settle whether each decision has a checkable acceptance criterion, or whether 'SKILL.md states X' is the intended and sufficient check."
  - "Relocation and exclusion candidates in decision-record.md: rationale narrative (e.g. the 2026-09-30 hand-back history), the Auto-mode assumptions that restate D2/D5, and the Open risks entry about the operator-owned HANDOFF file. Apply the completeness-before-leanness test before proposing any move to support-log.md or out of Discussion."
  - "Notes for the plan writer names specific functions and a method signature (`Runner.Send(guid, text)`, `runDriverSpawnAndWait`, `stepErrFields`). The section is optional, so its absence is never a finding; settle only whether this content exceeds the writer stencil's Step 2 bound on signatures and exact seams, which would make it an exclusion candidate."
---

Seed focus for round 1.
The design is autonomous (every question an auto-pick), so the entries weight internal consistency across D1–D12, the Auto-mode assumptions and the Acceptance criteria over gap-filling.
Checked against the rubric's Do-not-flag list: no entry asks for rejected alternatives in decision-record.md, for a Notes subsection, or for fuller call-site enumeration.
The last entry follows the symmetry rule: it questions content the writer stencil told the writer not to gather, not content the writer omitted.
