# Round 1 fixer report — discussion

## Changes

- F1, fixed as proposed: Stops by settler item 2 names `go-rule` records with action `restep` or `verify-retry`; a sentence says `moved`, `busy-exit`, `running-exit` and `hand-over` settle nothing; an acceptance criterion pins a `moved` record followed by an `orch` goto reading `orch`.
- F2, fixed by picking the leaf option: the settle record type, its stop key, settler and action names, and a byte-level tolerant decoder move from `internal/sheddriver` to `internal/shedrun`, which already declares the file's path and is on `reportengine`'s allowlist.
  The `PATTERN-told-geometry` Constraint names that move beside `internal/durfmt`, the docs Constraint adds `shedrun`'s and `sheddriver`'s docs, and the support log rejects importing `sheddriver` (it imports `lock`, `shedverbs` and `reedengine/render`, and `ReadSettleRecords` opens a file).
- F3, fixed as proposed: sleep stays inside phase time and is subtracted only from active, wait and attempt time.
- F4, fixed by accepting the pane format: durations of a day or more render as `2d03h`, without seconds, and the acceptance criterion says so.
- F5, fixed as proposed: a partial run adds to no bucket, and bucket totals clip to the window as well as the bucket.
- F6, fixed as proposed: the Go driver passes `shed` through `Deps.Goto` and `Deps.Step`; the `PATTERN-shed-driver` Constraint and the acceptance criterion say a driver resume writes `shed`.
- F7, fixed as proposed: the rationale is shortened, and the support log's Rejected alternatives gains folding `tackler` into `shed` or `driver`.

## Disputed

None.

## Deferred

None.
