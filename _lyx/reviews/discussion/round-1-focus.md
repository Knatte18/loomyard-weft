---
round: 1
focus:
  - "D9 versus Scope Out: the scan fails on any production `DisallowUnknownFields` call, while agent-written contract files stay strict. Settle whether any of those strict decoders is JSON and would trip the scan, and whether the Decision states the exemption."
  - "D2/D3 merge rule against the acceptance criteria: settle whether the case-variant criterion, the opaque-type rule and the element-unchanged comparison are consistent with each other, and whether any Decision lets a stale or mis-attached kept key through without a stated bound."
  - "D5 hang classification: settle whether the exit-code and log-marker rules are bounded for every way they can skip the rerun, and whether D5, D6 and the Open risk about a healthy over-window package agree on how a false hang reaches the fixer and the stuck reason."
  - "D7 watchdog edges: settle whether the inherited-slot exemption, the `-bench`/`-fuzz` exemption, the absent-versus-zero key rule and the SIGQUIT target selection each state what they let through and what bounds it."
  - "Relocation check: look for deliberation narrative in `decision-record.md` (for example D7's `proc-child-lifetime` rationale, or Goal/Scope restating Decisions) that belongs in `support-log.md`, extracting any requirement into Decisions or Constraints first."
---

Items are drawn from the rubric's attack-surface question, the completeness-before-leanness test and a consistency read across D1–D9.
None targets a `Do not flag` item; the stencil's ban on gathering signatures and `file:line` citations is respected, so no entry asks for them.
