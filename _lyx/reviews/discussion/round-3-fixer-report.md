# Round 3 fixer report

Every finding in `round-3-review.md` was fixed in `_lyx/discussion/decision-record.md`; `support-log.md` needed no change.

## Changes

- F1 (D7, Constraints, Acceptance): the held-run-lock refusal no longer names `lyx loom status`, which shows no lock holder for a halted run.
  Its way forward is a retry of `lyx loom resume` in a few seconds, and the bullet says it applies whatever the driver strand.
  The `PATTERN-refusal-way-forward` constraint reads "a retry of `lyx loom resume` while a lock holder finishes".
- F2 (D7, Acceptance): the no-live-driver refusal splits on reed's strand table, never the seed's driver.
  A dead driver strand names `lyx batten run <slug>`; no driver strand in state names `lyx loom start --no-attach`.
  The Acceptance row for the no-live-driver and running-with-neither cases carries both.
- F3 (D6, D8, Acceptance): D6 names the halt wait reason shown by `lyx batten status` as the carrier of the way forward, replacing its `lyx loom start` text with `lyx loom resume`, and `lyx loom start --no-attach` after a failed revive or with no driver strand.
  The awaiting hint follows, the notice line and channel stay unchanged, D8 adds `battenshed`'s `doc.go` sentence, and Acceptance has the reason in its batten row.
- F4 (Acceptance): the held-lock row drops "and no live driver", matching D7.
- F5 (D3): the deadline is a value `verifytree.Verify` is given, with production callers passing the 60-minute constant and a test a short value.

## Deferred

Nothing deferred.
