# Card 4 — reed directory reports a retiring strand

**Edit:**
- `internal/reedengine/directory.go`
- `internal/reedengine/directory_test.go`

**Intent:**
`DirectoryRow` gains `Retiring bool`, copied from `Strand.Retiring`, for both the session-up and the session-down rows.

`Directory` is the one reed read that answers without a live session (every row dormant when the session is down), which is exactly the case after a crash.
Both new readers of a child's driver strand need it: `lyx loom resume` (card 14) tells a dead, a retiring and an absent driver strand apart, and batten's restore (cards 16 and 17) does the same, and both must read a strand table whose session died with the machine.
`Status` cannot serve them, since it refuses with `ErrNoSession` when the session is down.

`directory_test.go` pins that a retiring strand's row carries the flag with the session up and down.
`lyx reed list` builds its envelope field by field, so its output is unchanged.

**ImpactSummary:** One added field on a read-only row type; `lyx reed list` is its only existing consumer.
