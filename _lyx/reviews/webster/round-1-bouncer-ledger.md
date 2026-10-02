---
round: 1
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
    rounds: [1]
    status: open
---

Round 1's reviewer approved with ten findings (F1–F10), and the Burler fixed all ten in eleven commits (`053643289`..`21a1a9783`).
The judge verified each fix against HEAD; build, vet, unit and integration tests pass.

F1–F3 reworked PR-Rework's round selection: only the highest round is considered, an uncommitted highest round is in flight and is re-keyed to a superseding rejection, records are written atomically, an undecodable record is an error, and a classed in-flight round goes straight to its commit.
F4 made the Plan-Review skip also require the working-tree plan to be exempt.
F5–F9 were small correctness and doc fixes.
F10 rewrapped the fixed-column Go comments.

The judge considered and did not raise one path: a classless round record at HEAD, left only by an operator hand-removing a pending rejection, is now re-keyed by a later rejection rather than opening a new round.

One finding is open: four new doc comments break mid-sentence at a comma that joins a list item or a compound predicate rather than at an independent-clause boundary (`internal/loomcli/parity_test.go:546`, `internal/loomcli/wiring_commitstatus_test.go:411`, `internal/loomcli/wiring_rework_integration_test.go:217`, `internal/loomshed/gates_test.go:437`).
