# Card 12 — loom status reports shuttle's waits

**Edit:**
- `internal/loomcli/reviewstatus.go`
- `internal/loomcli/reviewstatus_test.go`

**Uses:**
- `internal/shuttleengine/waitmark.go`

**Intent:**
Loom's `Waiting` hook reads, in order: the verify marker, the shuttle wait marker of the worktree's live runs, the parent-review note (D1).
The first marker that is present with a live pid wins; a marker with a dead pid is skipped, not the end of the search.

`reviewWaitingFor` gains the shuttle step between the two it has: it loads shuttle's config at call time with `shuttleengine.LoadConfig(anchorPath, "shuttle")`, the degrading loader loom's runner wiring already uses (`PATTERN-config-strictness`), and calls `shuttleengine.ReadWaitMarker` over the location's anchor path.
A live marker renders through card 10's formatter as `<producer>: <kind> running <elapsed>`, with the kind (`gate <entry name>` or `background shells`) as the what and no detail clause, e.g. `Plan-Write: gate validate-plan running 2m`.
A config load error is returned as the hook's error, as a verify-marker read error already is.

`reviewstatus_test.go` pins, under a fixed clock and a temporary shuttle run root: a live shuttle marker is reported when no verify runs, a live verify marker is reported ahead of it, a dead shuttle marker falls through to the parent-review note, and no marker with no review is empty.

**ImpactSummary:** One more read in loom status's waiting chain; the hook's signature and the other two reads are unchanged.
