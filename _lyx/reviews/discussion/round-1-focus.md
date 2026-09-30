---
round: 1
focus:
  - "D1 merge-only walk: settle whether the first-parent rule admits every shape `lyx fabric merge-in` actually produces (a merge commit whose first parent is the fork's commit), and whether any merge-in outcome can move HEAD without a merge commit (fast-forward, or a conflicted merge concluded through `merge --continue`) that the rule would then refuse, re-creating the wedge."
  - "D2 vs D1/D3 consistency: settle whether recording `CardSHAs`, the post-batch delta, and drift's triggering SHA at the report's `head_sha` while `DoneChecks` and drift's tree resolution read the merged worktree is a coherent pairing, or whether the decision record leaves the plan writer a contradiction to resolve (a drift finding triggered at one SHA but computed against another tree)."
  - "D4: settle whether the in-progress-merge refusal is stated in terms the plan writer can apply correctly in a linked git worktree and on the hub's warp/weft pair (which side's merge state is probed), or whether the decision leaves that choice open."
  - "D5/D6 anchor: settle whether the decision record says which anchor the `merge-in` RunE hands the D5 predicate, and whether that is the same anchor webster's `Dir` resolves `state.json`/`outcome.yaml` against for the worktree being merged into, so the warning fires for the run it is about."
  - "Weft side of the parent merge: `_lyx/webster/` is fabric-synced, so settle whether the decision record needs a decision or open risk on a parent merge-in touching webster's own durable state or reports in the task's `_lyx/`, or whether that is outside this task."
  - "Relocation/leanness: check `## Open risks` and `## Auto-mode assumptions` in `decision-record.md` for content that restates a rejected alternative's deliberation (e.g. the second-parent provenance risk) rather than a requirement the plan writer needs; apply the completeness-before-leanness test before proposing any move to `support-log.md`."
  - "Acceptance criteria vs Decisions: settle whether every decision (D1–D7) has a testable criterion and every criterion traces to a decision, in particular D3's warning content on the two-merge case and D6's predicate-error degradation path."
---

Seeded from a read of both artifacts, the writer stencil, and the code sites the record names (`recordbatch.go`/`recoverbatch.go` head cross-checks, `merge_verbs.go`, webster's `Dir`/`OutcomePath`).
Excluded per the rubric: the optional "Notes for the plan writer" section, rejected alternatives' absence from `decision-record.md`, and call-site enumeration, which the writer stencil also tells the writer not to gather.
