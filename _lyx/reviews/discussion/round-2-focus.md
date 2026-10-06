---
round: 2
focus:
  - "Decision D7 (the reset that updates the remote): settle what each failure leaves behind, namely a failed remote update after the local warp checkout is already reset, and a re-run of the reset; settle whether a re-run converges instead of refusing on the run's own commits."
  - "Decision D7: settle how a remote tip that already holds the run's own abandoned commits, with the local HEAD already rewound by an earlier reset, is handled, since that is the state the bug reports."
  - "Decision D7: settle whether the lease value is the same remote tip the ancestry check ran against, so a remote move between check and update refuses instead of overwrites, and whether the refusal names a way forward."
  - "Decision D6 (re-driving a restamped card): settle which terminal batch statuses of the batch state model it accepts and refuses, with `internal/websterengine/state.go` `BatchState.Status` as the source, and whether the acceptance criteria cover the same set."
  - "Decision D6 and D4: settle whether the stated way forward through `recover-batch` holds for every batch it names, given `internal/websterengine/recoverbatch.go` refuses a failed batch carrying uncheckable suspect paths."
  - "Decision D3 (relaxed Delete-target check at dispatch): settle what the relaxation lets through beyond its stated scope, what bounds that residue, and how the Rework-Plan-Gate treats a Delete target an earlier batch already removed."
  - "Decisions D1 to D4: settle whether the dispatch checks use one consistent card-scope phrase, and whether D4's reuse of the D2 check at record-batch and recover-batch time states the scope it needs."
---

Round 1 findings drive these entries; each restates the decision and source it depends on.
