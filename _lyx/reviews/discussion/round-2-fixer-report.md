# Fixer report — discussion round 2

## Changes

- F1 (fixed as proposed): Stops by settler now says the report reads two settle-record files at the tag, the loom run's and batten's, and states the join.
  A record names a stop when its producer and state equal the stop's and its history length equals the count of entries stamped at or before the stop's change, in the history its driver drives.
  A record matching no stop is a finding.
  New acceptance criterion: a loom-file `go-rule` `restep` record matching a batten child change reads `shed`, and a batten-file record names only a batten-row stop.
  Premise checked: `sheddriver.StopKey` carries producer, history length and state (`internal/sheddriver/stopkey.go`), and the two files come from `internal/battencli/drive.go:167` and `internal/loomcli/drivewiring.go:142`.
- F2 (fixed as proposed): Sleep, clipped to the run's span, is now left out of every row interval, and so out of phase, wait and attempt time.
  It is reported per run and per bucket, clipped to the bucket.
  Output names a sleep column in the run and bucket tables.
  The first acceptance criterion now reads phase seconds plus sleep sum to the span.
  Waits keeps active as span minus sleep minus waits, and the tokencount note is unchanged.
  Premise checked: `tools/tokencount/shed.go:49` and `:181` leave sleep out of every row.
- F3 (fixed as proposed): item 3 now reads a settle record with settler `tackler` and action `restep` naming the stop.
  A new sentence says a `spawn` record of any settler, a `parent` `hand-back` record and every `arbiter` record settle nothing by themselves, and an arbiter settle is read from its applied attempt in item 4.
  Premise checked: `internal/sheddriver/drive.go` writes those pairs at lines 452, 494, 540, 552 and 866.

## Disputed

None.

## Deferred

None.
