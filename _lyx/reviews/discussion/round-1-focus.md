---
round: 1
focus:
  - "D12 vs Scope: the invariant says every refusal a run can reach names its way forward and has a table row, but landing, batten, orch, fabric, burler and board refusals are out of scope. Settle whether the invariant as worded is violated on the day it lands, and whether its wording needs a scope or a transition clause."
  - "Ledger item 17 and the fourth Open risk: whether loom's fresh-start coherence gate may be re-entered through `goto` is left for the Plan to decide per gate. Settle whether this is a design decision the Discussion must make (the stencil says design the full scope now) or an enumeration the Plan may legitimately own."
  - "D4 parent-write split: a path can be both in the reports dir and git-ignored, or outside the tracked tree yet inside the reports dir. Settle whether D4 states which rule wins, so the Plan can classify every parent-write path unambiguously."
  - "D11 vs the Constraints' Shed Verb-Set bullet: `goto` validates the target against the recipe's producer names and resets a segment's bounce budget, while its body lives in `internal/shedverbs` and derives no path. Settle whether the record says how `goto` obtains the producer list and what bounds a 'segment', or whether that is left undecided."
  - "D5 verify re-run: settle whether 'a verify cannot run' and 'in-process on the worktree as it stands' are defined well enough to tell a verify failure (correctness, D6) from a transient I/O failure (D2's grouped transient row), and whether the D5-to-D6 escalation can itself wedge on a flaky verify."
  - "D6/D8 interaction: D6 marks parent findings seen when a batch fails, and D8 consumes each parent finding once. Settle whether a correctness-class parent finding (a Master write into the tracked tree) consumed at one batch can let a later batch record over the same unaudited content, and whether the record takes a position on it."
  - "Acceptance criteria, the per-row 'take the way forward and assert the run progresses' test: settle whether that is achievable for every class D2 leaves with per-row tests, including transient rows whose way forward is waiting on a lock holder or a plain git command."
  - "Relocation and leanness in decision-record.md: check whether the Auto-mode assumptions restate Decisions and Scope verbatim (verb names, the out-of-scope list), whether Rationale lines carry deliberation narrative that belongs in support-log.md, and whether the Notes for the plan writer carry code-level identifiers the writer stencil told Discussion not to gather. Apply the completeness-before-leanness test before any move."
---

Focus is on internal consistency between decisions, constraints and the new invariant, on the two items the Discussion deferred to the Plan, and on relocation candidates.
Call-site enumeration, the absent rejected alternatives in `decision-record.md`, and the optional Notes subsection's presence are excluded per the rubric's Do-not-flag list.
