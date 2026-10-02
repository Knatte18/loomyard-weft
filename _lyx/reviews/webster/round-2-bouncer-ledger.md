---
round: 2
ledger:
  - key: rework-superseding-rejection-archives-partial-plan
    rounds: [1]
    status: resolved
  - key: rework-torn-round-record-splits-round
    rounds: [1]
    status: resolved
  - key: rework-classed-resume-reruns-session
    rounds: [1]
    status: resolved
  - key: skip-seam-ignores-working-tree-burler-edits
    rounds: [1]
    status: resolved
  - key: empty-first-card-reads-as-one
    rounds: [1]
    status: resolved
  - key: describe-drops-prior-records-on-io-error
    rounds: [1]
    status: resolved
  - key: webster-archive-collision-way-forward
    rounds: [1]
    status: resolved
  - key: loomreworkdir-doc-says-appended-cards
    rounds: [1]
    status: resolved
  - key: overview-md-merged-sentences
    rounds: [1]
    status: resolved
  - key: go-comment-fixed-column-wraps
    rounds: [1]
    status: resolved
  - key: go-comment-non-clause-comma-breaks
    rounds: [1, 2]
    status: resolved
---

Round 1's reviewer raised ten findings (F1–F10), and the Burler fixed all ten in eleven commits (`053643289`..`21a1a9783`).
F1–F3 reworked PR-Rework's round selection and made round records atomic.
F4 made the Plan-Review skip also require the working-tree plan to be exempt.
F5–F9 were small correctness and doc fixes.
F10 rewrapped the fixed-column Go comments.

One residue stayed open after round 1: four doc comments broke mid-sentence at a non-clause comma.
Round 2's reviewer raised it as F1, and the fixer joined each sentence onto one line in `d7d5919dd`.
The judge verified the fix and re-scanned every comma-ended comment line in the range; all the other breaks come before an independent clause.

Every ledger entry is resolved, and the gate approved at round 2.
