---
round: 1
focus:
  - "Decision 5's shuttleengine change: evaluateGate today refuses Pending and Final from a non-pass_on_cap entry, and the Decision relaxes that for this entry. Settle whether the Decision states how the relaxation is keyed (this entry only, or any entry with some property) and what it can now let through, per the attack-surface rule."
  - "Decision 5's two counters: the recipe's `attempts: 3` and the store-counted rejected rounds. Settle whether the record says which one decides the cap, and whether the in-memory attempt count (which resets on attach) can still fail or pass the entry independently of the store count."
  - "Decision 5's `lyx loom review approve` supersede path and the reject-can-halt bound: settle whether the stated bounds cover who may call each verb, how 'a run halted at the cap' is determined mechanically, and whether the Acceptance criteria exercise the refusals the bound promises (approve outside the cap, approve on an earlier round, resume without approval halting again)."
  - "Decisions 3 and 4 together for a nested pair (Open risks, first item): the resolved reviewer is an orch that never runs, so parent-review holds until `parent_review_wait_min` and then passes with a Warn. Settle whether that is a guard made bypassable by this design, and if so whether the Decision that introduces it states the bound rather than leaving it only in Open risks."
  - "Decision 4's liveness seam: a reed engine built over the parent worktree's geometry, reached through GateConfig, and the 'legacy-name reviewer is treated as live' branch. Settle whether the Decision bounds what a failed or erroring liveness read does (hold, or prompt) and whether the told-geometry constraint on parentreview holds for it."
  - "Constraints, sixth bullet ('no Go process sends into a Claude session except the orch watcher as today'): check it against the parent-review gate's existing delivery and re-prompt sends to the writer, and settle whether the constraint as worded is accurate or needs narrowing."
  - "Leanness of the Decisions' Rationale paragraphs (notably 2, 4 and 7): settle whether any of them argue against a not-chosen option already recorded in support-log.md's Rejected alternatives, and apply the completeness-before-leanness test before proposing any move."
---

Seed rationale.
The design adds two escalation edges (reject can halt a run, approve can supersede a reject) and relaxes one engine guard (Pending/Final from a non-`pass_on_cap` entry), so most of the focus sits on Decision 5's bounds and on whether its cap has one counter or two.
Decisions 3 and 4 together change who reviews and when the gate holds; the nested-pair case turns parent-review into a timed pass, which the record currently names only as a risk.
The remaining entries check one constraint's accuracy against existing gate behaviour, and the Decisions' rationale for re-litigated alternatives.
Per the rubric and the writer's stencil, none of these ask for call-site enumeration, rejected alternatives in the decision record, or a Notes subsection.
