---
round: 1
focus:
  - "D6 live-pair rule: settle whether its cases are mutually consistent. A pair is live when the weft exists locally, yet the non-live paragraph says an archived remote weft is replaced and a diverged remote warp is refused. Which rule wins when a local weft exists and origin's weft is archive-tagged?"
  - "D6 and R3: D6 says an uncovered remote weft makes the pair live, and R3 admits that adopts a removed pair whose archive step failed. Does the Bound state what stops a stale warp branch from being adopted in that case, and does the Acceptance criteria cover the archived-weft refusal against a local weft?"
  - "D7 reconcile adopts an origin weft whether or not an archive tag covers it. Check that this guard bypass states what it lets through and what bounds it, and that it does not conflict with D6's archive rule for the same slug."
  - "D5 and A4: a failed origin probe or fetch now stops add, reconcile and checkout. Check the attack-surface question for the new failure mode: does reconcile's all-pairs run keep repairing other pairs when one pair's resolver errors, and is the skip-git and skip-push opt-out bounded?"
  - "D4 Finalize notice: check that the diff base (merge-base of the task's weft branch and its parent's weft branch) is well defined on the already-landed path, where the merge-base may already equal the tip, and on a stacked task. Does the Decision say what an empty or misleading diff means there? Also check that the Stuck-on-read-failure path leaves the parent untouched on both paths."
  - "D3 and D1 and D2: check that the three together close the reported bug without a gap. Is there a creation path or an existing worktree state that D1 and D2 miss, so the new ReadOriginFor error fires with no repair route? Check that the R4 reviewer-resolution behavior change is bounded."
  - "The 'Added after Discussion' section sits under Decisions. Check that it is a Decision plus Rationale the plan writer can act on, that its sequencing instruction does not contradict the Scope or Acceptance criteria, and whether any of its text belongs in support-log.md instead. Before proposing a relocation, apply the completeness-before-leanness test."
  - "Acceptance criteria against Decisions: look for a Decision whose observable behavior has no criterion, for example D2's Detail text, D3's Remove fallback, D6's local-fast-forward rollback and D7's checkout rollback. Also look for a criterion that asserts something no Decision states."
---

Seeded from a read of both artifacts, the rubric's Do-not-flag list and the discussion stencil's gather limits.
Exact signatures, call-site enumeration and file:line citations are not focus items; the stencil tells the writer not to gather them.
