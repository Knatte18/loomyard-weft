---
verdict: APPROVED
---

No findings.

The diff makes the change the board entry asks for.
The twill directive and the fixer self-check state the wait as a hard rule in their wait step, and the cluster handler holds the review file until the last fork has returned.
`internal/twilldirective/doc.go` and `internal/burlerengine/doc.go` describe the rule.

Focus directive verdicts:

- Scope: every stencil under `contracts/stencils/` that mentions forks was checked.
  The discussion chair's lens review renders through `twilldirective` and so picks up the rule.
  The orch, bouncer and shed stencils spawn one-shot single forks and judge no set of lens findings.
  Webster's implementer forks are excluded by the decision record.
  `burler-cluster-solo.md` spawns no forks.
  No omission is unjustified.
- Stencil wording: the relaunch and resume rules for lost forks, the merge-duplicates judging and the host-neutral judge parts are untouched.
  A lost fork never returns, and the relaunch part already names it "no report", so the wait rule does not contradict it.
- Cluster fan: "writes nothing to the review file until the last fork has returned" fits the stencil's consolidation paragraph, which writes the ONE review file after the forks, and matches phase (3) of the cluster paragraph in `internal/burlerengine/doc.go`.
- Change description: each claim in `_lyx/landing/summary.md` is borne out by the diff, and the `internal/burlerengine/doc.go` reflow changes no rule beyond the new one.

## Rejected

- The cluster-fan stencil has no lost-fork rule, so a handler resumed after losing its forks has no instruction on whether to write: the old text was equally silent, so the diff does not introduce the gap and the entry does not ask to close it.
