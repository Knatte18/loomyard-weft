---
verdict: APPROVED
---

## Scope against the board entry

The board entry (issue #722) asks that a hold refresh stop writing a `held` event every couple of seconds: one `held` when a hold starts, and one event when it ends or changes, not one per refresh.
The decision record has no `Added after Discussion` entries, so the board body is the measure.

The diff of PR 727 at `b94787ef3` against `main` touches only `internal/creelengine`:

- `Store.Hold` journals a same-owner renewal as a new `renewed` record instead of `held` (`verbs.go`).
- The fold applies `renewed` to the hold like `held` but returns before adding a log entry (`fold.go`), so `creel log` no longer shows renewals.
- A hold taken after expiry or release still journals `held`, and release still journals `released`, so a start, a change of owner and a release each show once.
- The package doc's Holds section describes the new event (`doc.go`).
- The existing hold row is extended to cover a renewal surviving a reopen, the logged events `held, released`, and a retake after expiry logging `held` again (`verbs_test.go`), and the retention test pins a `renewed` record across the carry (`journal_test.go`).

That covers the entry's ask and adds nothing it does not call for.
A hold that lapses by expiry was not logged at its moment before this change and is not logged now; its end shows as the next `held` start.
The decision record reads "ends or changes" as a release, an expiry followed by a new start, or a new owner, and the entry's complaint is the per-refresh spam, so I do not count a separate expiry event as a point the change leaves out.

## Open risks carried over

The record carries one open risk: an older binary folding a journal with `renewed` records ignores them and keeps a hold's first expiry.
It names no unrecorded parent correction, so it does not escalate.

## Overlap with peers and parent commits

`lyx loom peers` lists no parent commits and four peers: `config-migrate-hub-layer` and `timing-report-darn` (row 17), and `batten-start-speed` and `orch-rules-files` (row 9).
None of their changed paths touch `internal/creelengine`, and this run changes no shared bookkeeping file such as `docs/overview.md` or `PATTERN.md`.
There is no overlap, so no landing order applies.

## Superseded entries

None: no other board entry covers creel hold logging.
