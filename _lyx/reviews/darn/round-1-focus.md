---
round: 1
focus:
  - "Scope: the board entry's last sentence asks the same check for any other stencil where a session spawns review forks and acts on their findings. Grep `contracts/stencils/` (twill, burler, edit, friction, and others that mention forks) for such a stencil the diff leaves out, and settle whether each omission is justified by the summary's design decisions."
  - "Stencil wording: in each of the three changed stencils, compare the diff against the prior text. Settle whether any existing rule (relaunch and resume handling of lost forks, merge-duplicates judging, host-neutral twill judge parts) is dropped, weakened or contradicted by the new wait rule."
  - "Cluster fan: in `contracts/stencils/burler/burler-cluster-fan.md`, settle whether 'writes nothing to the review file until the last fork has returned' is consistent with the rest of that stencil's steps and with what `internal/burlerengine/doc.go` now says."
  - "Change description: check each claim in `_lyx/landing/summary.md` against the diff, including that the doc.go reflow in `internal/burlerengine/doc.go` changes no rule beyond the new one."
---

Seed focus only; the reviewer still reads the whole diff.
