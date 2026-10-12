---
verdict: APPROVED
---

The plan was read against `_lyx/discussion/decision-record.md` alone: no `_lyx/loom/rework/` directory exists, so this is a generation-0 plan.
`lyx loom validate-plan --overlay` with `{}` reports `ok: true`, with informational `create-new-unit` findings only.
No fix is proposed, so no overlay run with fixes was needed.

## Focus verdicts

- **Card 21, `pauseTick`, a watcher running when the entry is written (`--grace 0`, or `poll_interval_ms` at least the grace): not a defect.**
  The past-grace rule fires only for an entry whose `GraceEnd` lies before the watcher's first tick, which the card keeps in memory as the watcher's own start.
  A watcher already running has a first tick before `PausedAt`, and `GraceEnd >= PausedAt`, so the rule cannot fire for it.
  The entry goes to the "at or after `GraceEnd`" bullet even when the first tick that reads it lands after `GraceEnd`, as the card says in so many words, and a busy orch is recorded and interrupted.
  The test row "a watcher created before the entry, with the entry written at grace 0 and the first tick reading it after `GraceEnd`, interrupts a busy orch and records it" reaches that case.
  The rule keys on the watcher's first tick, not on the entry's first read.
- **Card 21, an orch started between `PausedAt` and `GraceEnd`: not a defect.**
  `State.LaunchedAt` is set by `ResetForFreshLaunch` from the `launchedAt` its callers already pass (`internal/orchcli/start.go:252` and `:302`, `internal/orchengine/watcher.go:369`).
  The bullet "an entry whose `PausedAt` is not after `st.LaunchedAt` ... nothing is interrupted, nothing is recorded and no resume line goes" covers such an orch, matching D4's `lyx orch start` row and the acceptance criterion.
  The test row "a state whose `LaunchedAt` lies between the entry's `PausedAt` and `GraceEnd`, with a busy probe, gets no `Interrupt` call at the grace end and no resume line after the resume" reaches it.
- **Card 23, sandbox scenario S12: not a defect.**
  `tools/sandbox/SANDBOX-CORE-SUITE.md`'s scenario headings run S0 to S11 (`### S11 -- Routine cleanup as a dry run`), and its Session log format block lists `S0:` to `S11:`, so S12 is unused in both.
  Card 25 edits S8 in place and adds no number.
  `resume` is a root command in `newRoot()`, so the `**Covers:** pause, resume` tag passes `cmd/lyx/sandbox_coverage_test.go`'s drift check that every covered token names a registered module.
- **Card 17, `battenCLI.wire`'s `pauserecord.SetGateSlug` call: not a defect.**
  `TestWire_PointsTheNetworkGateAtItsSlug` installs a gate with an empty slug over a record holding a live entry for the wired slug only, wires through `battenCLI.wire` as `wiredPrime` does (`internal/battencli/wire_seams_test.go:119`), and asserts `Sleep` ran inside `pauserecord.Network`.
  With the call dropped, the gate's slug stays empty, `Record.Covering("")` is satisfied by a hub entry only (card 1), `WaitUncovered` returns at once, and `Sleep` is never called, so the test fails.
  The test is not parallel, and Go runs the file's `t.Parallel` wire tests, which also call `SetGateSlug` through `wiredPrime`, only after the sequential tests, so they do not race its installed gate.

## Rejected

- `orchcli#orchCLI.adoptStrand` passes `time.Now()` to `ResetForFreshLaunch`, so under card 21 an adopted session whose recorded strand differed from the live one would read as launched after a live entry's `PausedAt` and go uninterrupted.
  It is not raised: `State.Strand` changes only through `ResetForFreshLaunch` (`internal/orchengine/state.go:288`), so a live strand differing from the recorded one comes from a lost or hand-edited state file, not from any flow D4 or D7 describes.
