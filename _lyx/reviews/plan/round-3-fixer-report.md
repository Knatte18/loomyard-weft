# Plan-Review round 3 — fixer report

All four findings were checked against the code and the decision record, held, and were fixed.
`lyx loom validate-plan` over the fixed plan reports `ok: true`, with informational `create-new-unit` findings only.

## Fixed

### F1 — hubgeom's package doc goes stale under card 5 (MEDIUM)

`internal/hubgeom/doc.go` lists its tellers as "ReedGeometry, BurlerGeometry, and WebsterGeometry", then `ReconcileGeometry`, and no card edited it.
Card 5 now lists `internal/hubgeom/doc.go` under `**Edit:**`, and its Intent adds `GateSlots` to that paragraph.

Per the finding's method fix, I checked every existing package a card changes for a `doc.go` or package-comment sentence the change makes false:
`verifyrun`, `configreg`, `configsync`, `websterengine`, `landingshed`, `loomshed`, `shedrecipe`, `shedadapters`, `planparser`, `impactset`, `configcli`, `hubgeom`, and `docs/overview.md`'s entries.
One more sentence goes false: `internal/websterengine/doc.go` says "See geometry.go for what each of Geometry's eight fields means".
The struct already has more fields than that, and card 5 adds two more.
Card 5 now also lists `internal/websterengine/doc.go`, and its Intent drops the count from that sentence.
The other packages' sentences stay true or are already rewritten by a card: 4, 9, 12, 14, 15, 17, 18, 20, 21 or 23.
`verifyrun`'s import pin stays true, as card 3 states.

### F2 — the failure-record Bound omits the log-path redirect (LOW)

Card 21 confines the log path to the verify directory, which sits in the pair's own `.lyx/`, so an edited record can point at any file there.
Card 22's Bound now names this as a third thing an edited record lets through: another file in the verify directory, whose content a pair session can rewrite, as it can the log copy itself.
That route is bounded by the reviewer already reading pair-authored content and by Publish's full rerun.
Card 21's Bound now names the same route, notes that the gate never reads the log and only the review prompt carries the path, and keeps Publish's full rerun as the bound.

### F3 — card 21 names no file for its integration-tier rows (LOW)

Card 21 now creates `internal/loomshed/publishfailure_integration_test.go`, one table-driven test whose rows need a scratch repository: a real merge commit's full hex name is kept, and a non-merge commit's is dropped.
`publishfailure_test.go` stays tier 1 and keeps the rows that need no repository: `HEAD --output=x`, a short hash, a test-name metacharacter, a log path outside the verify directory, and the command rows.
The card now states that the `internal/loomshed` budget row rises by two; it already listed `test-budget.yaml`.
`internal/loomshed/testmain_test.go` is untagged, so the new file runs through the package's existing test main.

### F4 — card 11 leaves the one-marker premise unstated (LOW)

Card 11's Intent now states the premise.
Webster's gate, the round gate, Publish and Finalize are rows of one loom run and step one at a time, so the one marker per worktree carries every `verifytree.Verify` wait that can occur in a run.
Every other waiter writes its own wait record.
The Intent also states that an operator's `lyx webster verify` running beside a loom verify shares the marker and can hide the other's wait.
That is today's running-marker behavior and outside the guarantee.
This was checked against `verifytree.NewPaths`: every verify site writes `running.yaml` in its verify directory, and `loomcli/reviewstatus.go` reads that one path.

## Disputed

None.

## Deferred

None.
