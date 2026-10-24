---
round: 1
ledger:
  - key: numbered-orch-thread-refused-task-creation
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: taskguard-slug-free-non-orch-row-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: board-skill-omits-task-creation-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: note-step-doc-comment-overstates
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings.
F1 (design, MEDIUM) is the only gating one: the orch exemption must use `agentname.MatchesRole` so `orch-2` passes.
F2 asks for a slug-free non-orch test row, F3 for a board skill paragraph on the new refusal, and F4 for a doc comment rewording.
All four fixes are local and fixed by the finding text.
