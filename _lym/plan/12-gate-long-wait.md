# Card 12 — gate-long-wait

**Create:**
- `internal/gatecli#longWait`
- `internal/gatecli#TestLongWait`
- `internal/gatecli#chooseCap`
- `internal/gatecli#TestChooseCap`
- `internal/gatecli#helperLongCapEnv`
- `internal/gatecli#helperMaxRerunsEnv`

**Edit:**
- `internal/gatecli#runFront`
- `internal/gatecli#frontCall`
- `internal/gatecli#runGateHelper`
- `internal/gatecli#TestGateTest_DetachedRun`
- `//internal/gatecli/doc.go`

**Uses:**
- `internal/gateslot#ReadRunRecords`
- `internal/gatecli#runLogDirOf`
- `internal/shuttleengines#KeepWarmAnswer`

**Intent:** D5: a `--gate` key whose runs outlast the break-even waits in one long call instead of many short ones.

`frontCall` carries the call's gate key (the `--gate` value, empty without one), the resolver's `LongCap` and `MaxReruns`, and where its run records are (`runLogDirOf(location, inHub)`).
Before it counts its deadline, `runFront` takes its cap from `chooseCap(call, gateslot.ReadRunRecords)`: with a gate key and `MaxReruns > 0`, it reads this machine's records through the told reader, and when `longWait(records, key, cap, maxReruns)` holds it answers `LongCap`, 55m from the claude engine, in place of the cap; a read error, no key or `MaxReruns` 0 keep the short cap without a read in the last two cases, and so does a resolver fallback, whose `MaxReruns` is 0.
The chosen cap is the front's deadline and so its lease, so a long-cap front keeps the run going for its own deadline plus the grace.

`longWait` groups the records of `key` by `CallID`, leaving out records without one, and keeps a call only when it ran every tier it listed to an outcome: for each tier of its `TierList` a non-build record of that tier exists, and no record of the call carries `Carried` or `Abandoned` or is an empty-scope tier, a non-build, non-carried record whose `Command` and `BlockedBy` are both empty.
A tier the failed build blocked also has no `Command` but carries `BlockedBy`; it ended to an outcome, failed, so a call whose build failed counts.
So a carried call, an empty-scope call, an abandoned call (a call cancelled in its last listed tier or in the build step before it is one, since its records after the cancel carry `Abandoned`) and a refused call or one cancelled in a slot wait before its last tier (missing a listed tier's record) drop out.
A call's wall time runs from its earliest start, a record's `WaitStart`, or its `Start` where `WaitStart` is zero as on a build record, to its latest `End`.
It takes the latest five kept calls by that earliest start; with fewer than three it answers false; otherwise it answers whether their median wall time exceeds `capWait` times `maxReruns`.
Bound: an estimate from a few earlier calls can mis-pick, which costs cache money only, never a verdict.
The doc's Capped call section gains the long wait.

Tests:
- `TestLongWait`, tier0, a table of built records: three full calls of 30s against a 1s cap and 10 re-runs answer true; the same against 20 re-runs answer false; two full calls answer false; a call with a carried tier, one with an empty-scope tier, one with an abandoned record (in its last tier, and in its build record) and one missing its second listed tier's record are each left out, so three full calls plus those answer as the three alone; a call whose build record has a zero `WaitStart` counts its wall from the build's `Start`; a call whose build failed, its tiers blocked with `BlockedBy` set, counts as a full call; records of another key and records without a call id are ignored; only the latest five calls count, so two old slow calls under five new fast ones answer false.
- `TestChooseCap`, tier0, over a `frontCall` with a 1s cap and a 9s long cap and a fake reader that records its calls: three slow full calls of the key answer 9s; no gate key answers 1s with no read; `MaxReruns` 0 answers 1s with no read; a reader error answers 1s; a fallback answer (`MaxReruns` 0, `LongCap` equal to the cap) answers 1s.
- `TestGateTest_DetachedRun`, tier1, gains a row: with three full `finalize` calls of 10s each seeded through `gateslot.WriteRunRecord` into the hub's run directory, a 1s cap, `helperLongCapEnv=6s` and `helperMaxRerunsEnv=2`, which the gate helper now puts into its resolver answer beside `helperCapEnv`, a `--gate finalize` call whose stand-in runs 2s ends with the run's result on its first call instead of exit 75.

**ImpactSummary:** the front's cap choice reads this machine's run records for `--gate` calls only; every other call keeps the short cap.
