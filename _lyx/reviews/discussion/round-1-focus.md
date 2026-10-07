---
round: 1
focus:
  - "D2 command order and `go list -tags integration`: the round command vets and lists deps only under the `integration` tag, while the plan verify vets `tmux` and `llm` too. Settle whether a change to a `tmux`- or `llm`-tagged file or its reverse deps can pass the round gate uncompiled, and whether D2 states what bounds that."
  - "D5 `//lyx:guard`: an unmarked repo-scanning test is silently never run by the round gate, and the only stated control is review discipline. Settle whether the Decision states what a missed marker lets through and what bounds it, beyond the Publish catch-all."
  - "D2 fallback list versus the Acceptance criteria: settle whether every fallback condition and the empty-set behavior in D2 has a matching criterion, and whether the two agree on the conditions."
  - "D3 and D2 base lookup: the base is the latest plan-verify pass from per-command records. Settle whether a record written for a different tree state, an unreachable commit, or a missing record all land in a stated fallback, and whether the record's eviction or retention rule is bounded."
  - "D7 lint finding rule versus D6 allowed breaks: settle whether the mechanical finding definition and the stated bound agree with D6 on which breaks are legitimate, and whether the card-gate and round-gate diff scopes leave a gap (untracked files, renamed files, deleted lines)."
  - "D4 `publish_verify`: a repo-configured shell command run at Publish. Settle whether the Decision states what it can run and what bounds it, and whether the ordering with the D4 requirement that the `tmux` tier be green is a constraint the plan writer receives in `decision-record.md`."
  - "D1 against D2 to D4 for contradictions: settle whether the gate table, the per-decision text, the Scope list and the Acceptance criteria describe the same gates, in the same order, with the same commands."
  - "Relocation: settle whether any Rationale, Open risks or Auto-mode assumptions content in `decision-record.md` is deliberation narrative with no requirement for the plan writer. Before proposing a move, extract any requirement or constraint it carries into Decisions or Constraints, since Plan-Write never reads `support-log.md`."
---

Seed focus for round 1.
The entries steer attention toward weakened guards, unbounded edges and cross-section consistency.
Each stays inside the rubric: none asks for call-site enumeration, rejected alternatives in the decision record, or a "Notes for the plan writer" addition.
D8's open seam is delegated to Plan-Write on purpose, so the writer stencil's ban on gathering interface shapes means its absence is not a finding.
