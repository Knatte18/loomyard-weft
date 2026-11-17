# The dobby tears down or reports leftover pairs

Pairs that no run's own teardown takes no longer stand on disk unreported.
The dobby's ended-run teardown becomes a teardown of every leftover pair, judged from the run FSM,
and the pairs it must keep are reported.

## What the dobby tears down

`internal/dobbyengine/leftover.go` replaces `ended.go`.
Each tick classifies every listed run, done runs included, through `leftoverKind` into a `LeftoverKind`:

- `landed`: a run in the FSM state `landed` whose pair is still on disk.
- `cancelled`: a run in the FSM state `cancelled` whose cancel did not keep the pair.
  Its batten record goes in the same step, through `pairteardown`'s existing cancelled-record removal.
- `ended`: the old stranded-run case, a run whose board entry ended while batten left it short of its teardown.
- `handed_over`: the local pair of a run this machine's hand-over mark names while the registry reads another machine as its owner.
  This kind ignores pauses and holds, since the local removal takes no origin step.
  It also covers a hand-over inside the pair, which leaves the pair for the parent's dobby to remove.
- `orphan`: see below.

Apart from `handed_over`, a pair is torn down only with no pause, no Bolt hold and no other machine owning the run.
A run with a pass in flight, a held driver lock or a cancel under way is never taken.
A machine that adopted a run back owns it, so its pair stays.

`Deps.TearDownEnded` becomes `Deps.TearDownLeftovers`, which takes `[]LeftoverPair`.
The retry and backoff logic carries over unchanged.
In `internal/beamcli/dobbydeps.go`, `tearDownLeftovers` dispatches by kind:

- `ended` goes through the hub origin reconcile, as before.
- `landed` and `cancelled` go through `pairteardown`'s `Run` with `Remote` set.
  `landed` sets `Landed` so the claim is marked done.
  `cancelled` sets `Force`, since the cancel chose the removal.
- `handed_over` goes through the new `pairteardown.RemoveHandedOverPair`, in local mode.
- `orphan` goes through the new `pairteardown.RemoveOrphanPair`.
  It archives the pair's tips under owner archive tags, uncommitted files included,
  then removes the pair and leaves its branches on origin.

After each batch, `fabricengine.RegenerateHubWorkspace` runs, so the hub workspace file no longer lists a removed pair.
A local removal on its own only tries the workspace lock once.

The done-run clear now skips a run whose pair still stands.
Without that skip, it would remove a landed run's records before the leftover teardown read them.

## Orphan pairs

Only the prime's dobby reads orphans, through `battencli.DobbyOrphans`, on each probe tick.
An orphan is a task pair on disk that no board entry and no batten run directory on the prime names.
Its first sight posts one notice saying when the pair goes.
The time goes into a machine-local record at `fabricengine.OrphanFirstSightPath`,
under the hub `.lym` in the new `lymdirs.OrphanDirName`.
That way a dobby restart neither resets nor skips the grace.
An orphan is torn down once `dobbyengine.OrphanGrace` (24 hours) has passed since its first sight
and since the newest commit on either branch (`fabricengine.PairLastCommit`),
and only while no process registered under its slug lives.
A pair that stops being an orphan, or is torn down, leaves the record.

## Kept pairs

A cancelled run whose cancel kept its pair stays.
Its batten record keeps it out of the orphans even after its board entry is removed.
`RunView` gains `PairKept` and `KeptSince`, and `View.Kept` lists the pairs that still stand.
`lym beam status` renders one line per kept pair with its age and `lym fabric remove <slug>`.
Each fresh dobby start, never a re-exec, posts one notice listing the kept pairs and naming the same verb.

A run cancelled with keep before cancel intents existed has a batten record in shed state `cancelled` with no intent.
`runfsm.Derive` reads such a run as neither cancelled nor kept.
Each dobby process therefore runs `Deps.MarkKeptCancels` once before its first passes.
That is `battencli.MarkKeptCancels`, which writes the new `shedrun.WriteKeptCancel`,
a finished keep cancel intent naming this machine, for each such run of its worktree whose pair stands here.
It then commits and pushes the record through the synchronous records push, the same path `lym batten cancel` uses.
The kept-pairs notice waits until this step has succeeded.

## Tender resume

A run's transition now pushes synchronously, so `lym tender resume` no longer spawns a pair push for every pair it lifts.
`tenderengine.Deps.PushDeferred` (new) gates the pair push.
`tendercli` fills it with `fabricengine.PairPushDeferred`, which reads the pending flag that a pause-deferred detached push leaves behind.
A deferred anchor-change push is still paid.
This change is its own commit.

## Design decisions

- The kinds travel to the hub wiring with each slug, so one seam and one in-flight teardown cover every kind.
  The earlier seam took bare slugs and ran only the origin reconcile.
- The hand-over kind skips the pause and hold checks, as D7 asks for local mode.
  It still waits for a pass in flight, a held driver lock or a cancel under way.
- An orphan's teardown keeps its branches on origin (`Remote` unset).
  Its local tips that origin lacks go under pushed owner archive tags first, so "its branches archived" holds either way.
- `PushDeferred` is a new engine seam rather than a filter inside `tendercli`'s `PushPair`.
  That way the resume report's `spawned` list names only pushes that actually started.
  A nil seam keeps the old behaviour for callers that set none.

## Tests

- `internal/dobbyengine/leftover_test.go` replaces `ended_test.go`.
  - `TestLeftoverKind` adds rows for landed, cancelled with and without keep,
    a handed-over run under a hold and a pause, a run adopted back, and the hold and owner exclusions.
  - `TestTearDownLeftovers` carries the old retry and in-flight cases and adds a landed run that batten reads done.
  - `TestKeptPairs` pins the one notice per fresh start, its wait on the kept-cancel marking, and the view's kept list.
  - `TestOrphans` pins four behaviours: the notice and the teardown no earlier than the grace,
    an orphan with a recent commit or a live process staying,
    the first sight surviving a restart,
    and the record dropping orphans that are gone.
- `internal/shedrun/intents_test.go` pins `WriteKeptCancel`: it writes only over a cancelled record with no intent.
- `internal/tenderengine/resume_test.go` pins that a lift pushes only a pair whose push a pause deferred.

The hub-level wiring in `battencli` and `beamcli` (orphan listing, kept-cancel marking, teardown dispatch) has no new integration test.
The engine tests cover its decisions through the seams.
