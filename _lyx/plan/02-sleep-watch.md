# Card 2 — sleep-watch

**Create:**
- `internal/pauserecord#SleepTolerance`
- `internal/pauserecord#SleepWatch`
- `internal/pauserecord#SleepWatch.Observe`
- `internal/pauserecord#SleepWatch.Sleeps`
- `internal/pauserecord#AwakeSince`
- `internal/pauserecord#MonotonicAwake`
- `internal/pauserecord/sleep_test.go`

**Edit:**
- `internal/pauserecord/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Add the one shared sleep detection of D9, in `sleep.go`, which batten (card 16), the shed loop watchdog (card 19) and shuttle's Wait (card 8) read slept time through.

- `SleepWatch` keeps the previous wall and awake readings and the sleeps it detected.
  `Observe` records a reading; from the second on, the wall delta less the awake delta, taken from `UnixNano` values so no monotonic part hides a sleep, is a sleep when it exceeds `SleepTolerance` (a shorter lag is drift or scheduling delay).
  A detected sleep is recorded as the interval `[wall - gap, wall]`: it ends at the reading that saw it and is placed as late in the check interval as it can lie.
  Sleeps whose end is older than `Retention` before `wall` are dropped, so a long-lived watch stays bounded.
- `Sleeps` returns a copy of the recorded intervals.
- `AwakeSince` returns `now.Sub(at)` less `Excluded` over the union of every `excluded` list on `[at, now]`, never below zero.
  Because a sleep is an interval rather than a running total, an activity read at any time keeps exactly the slept time that lies after it: this is the fix card 16 applies to batten, whose accumulated total was dropped whenever the newest activity stamp moved, so an activity written between the last check before a suspend and the suspend lost the whole night.
- `MonotonicAwake` returns a reading of Go's monotonic clock from an origin taken at the call; it stops during suspend on Linux and macOS and keeps running on Windows, as batten's deps document today.

Tests, in `sleep_test.go`: one table test over `Observe` and `Sleeps` with batten's `TestSleepGap` rows carried over (hours of wall against seconds awake, an excess at and just over the tolerance, equal deltas, wall behind awake) plus a first reading that records nothing and a sleep older than `Retention` that is dropped; one table test over `AwakeSince` with an activity before a sleep (the whole sleep subtracted), an activity after it (none subtracted), an activity inside a pause interval overlapping a sleep (the overlap subtracted once) and no intervals.
`doc.go` gains the sleep detection paragraph; the budget row grows by the two tests.

**ImpactSummary:** Adds sleep.go to the new leaf package; the package doc and its budget row change, no caller yet.
