---
round: 1
focus:
  - "D5 (caller scan skips nested modules): a skipped directory is a guard removed. Settle whether a nested module can hold a caller of a root-module subject, and whether D5 states what that lets through and what bounds it."
  - "D2 and D4 (type-load fallback, name-only methods informational): settle whether each stated bound holds against the acceptance criteria, in particular that a package-level caller never weakens on the fallback path."
  - "D3 against D1 and the acceptance criteria: settle whether the fallback's import-path resolution, dot-import and alias handling, and the no-`go.mod` case agree with what the acceptance criteria demand, and that no criterion contradicts a Decision."
  - "D6, D7 and D8 consistency: settle whether the nested-module derivation (on-disk plus `Create` through the card, minus `Delete`, innermost module wins) gives the same answer in the card gate and in the check, including the whole-plan form for the overview."
  - "D8 bound: settle whether the stated misses (variables, `bash -c`, subshell, import-path argument) leave an unbounded way to pass a command the card gate would reject, and whether the check blocks anything legitimate."
  - "D9 and D11: settle whether the rubric rule that a blocked fix is not a legitimate finding can suppress a real defect the gate's checks do not catch, and whether D11's CLAUDE.md line has a stated bound."
  - "Relocation and exclusion: settle whether anything in Auto-mode assumptions, Open risks or Notes for the plan writer holds a requirement the plan writer needs that is not also in Decisions or Constraints, and whether any deliberation narrative in decision-record.md belongs in support-log.md."
---

Round 1 looks first at guards and edges the Decisions add, remove or downgrade (D2, D4, D5, D8, D9), then at cross-Decision consistency between the Decisions and the acceptance criteria.
The `Do not flag` list and the writer stencil's exclusion of exact signatures, `file:line` citations and exhaustive pattern research apply to every entry.
