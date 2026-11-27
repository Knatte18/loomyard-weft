# Card 6 — gate-hold-engine

**Create:**
- `internal/dobbyengine#GateHolder`
- `internal/dobbyengine#GateHoldDeps`
- `internal/dobbyengine#daemon.gateHolds`
- `internal/dobbyengine#gateHoldLine`
- `internal/dobbyengine#underWorktree`
- `internal/dobbyengine#gateHoldProcessMax`
- `internal/dobbyengine#TestGateHolds`

**Edit:**
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#daemon.tick`

**Extends:**
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/dobbyengine#RunWatchState`
- `internal/dobbyengine#runNoticePrefix`
- `internal/dobbyengine#Config`
- `internal/dobbyengine#newPassHarness`
- `internal/dobbyengine#passHarness.tick`
- `internal/dobbyengine#passHarness.advance`
- `internal/dobbyengine#passHarness.restartDobby`
- `internal/dobbyengine#loopClock.suspend`
- `internal/proc#ProcessSample`

**Intent:**
The gate-hold notice of D6, in a new `gatehold.go`; the hub wiring comes in card 7.

`GateHolder` is one held slot as the dobby reads it: `Slot int`, `Worktree`, `Site`, `Tier`, `Marker string`, `PID int`, `Started time.Time` and `TierTimeout time.Duration`, the tier's `timeout_min`, zero for a holder with no tier or a tier with none.
The package declares its own type, since `gateslot` is not among its allowed imports.
`Deps` gains `GateHolds GateHoldDeps`, which holds:
`Holders func() ([]GateHolder, error)`, the live holders with their tier timeouts, failing on a failed gate config load or a failed `Pool.Holders` read;
`TaskWorktree func(slug string) (string, error)`, the run's task worktree path, resolved from the slug alone;
`Tree func(pid int) ([]proc.ProcessSample, error)`, `proc.TreeSample` in production;
`MarkedPIDs func(entry string) ([]int, error)`, `proc.PIDsWithEnv` in production;
`Post func(ctx context.Context, slug, line string) error`, which posts under the gate-hold topic and the run tag.
A nil `Holders` or `Post` checks nothing.
`daemon` gains the episode sets `gateHoldReads` and `gateHoldPosts`.

The step: `tick` calls `d.gateHolds(ctx, now)` right after `d.runWatch` in the scheduling block, and it runs only on probe ticks, so `Holders` is called at most once per probe tick.
It sits in the scheduling block because it matches holders against `d.listedRuns`, which only the scheduling pass step lists.
Bound: while an anchor move, an adopt hold or an owed catch-up holds the scheduling, the check waits with it, and a hold past its bound is noticed on the first probe tick after the hold on scheduling ends.
It loads the config through `Deps.LoadConfig`; a failed load, or a failed `Holders`, logs one Warn per failure episode through `gateHoldReads` and skips the check for the tick.
`GateHoldFactor` 0 sends nothing.
Holders are grouped into holds by `PID`; a hold's start is the earliest `Started` among its slots, and its key is `<pid>|<start in Unix nanoseconds>`.
A hold belongs to a run of `d.listedRuns` when its first holder's `Worktree` is the run's `TaskWorktree` or a path inside it, compared on whole path segments by `underWorktree` after `filepath.Clean`, so `/hub/task-b` is not under `/hub/task`; a failed `TaskWorktree` skips that run with a Warn through `gateHoldReads`.
The bound is `GateHoldFactor` times the hold's largest `TierTimeout`, or `GateHoldMin` minutes when no holder of it has a positive one.
A hold is past its bound when `pauserecord.AwakeSince(start, now, State.Sleeps)` reaches it.
A hold past its bound whose key the run's `Watches` entry's `GateHolds` lacks is noticed once: `Tree(pid)` gives the holder's tree; when its `Marker` is set, `MarkedPIDs(Marker)` gives the marked PIDs, and each one outside the tree is sampled through `Tree(pid)`'s own entry; a failed `Tree` or `MarkedPIDs` still posts, naming the tree as unreadable.
`gateHoldLine` renders `[dobby notice] <slug>: a gate slot is held past its bound: slots <n,...>; tier <tier or none>; site <site>; started <UTC RFC 3339>; bound <bound>; holder pid <pid> <executable> cpu <cpu>`, then `; child pid <pid> <executable> cpu <cpu>` per descendant and `; outside the tree, carrying <marker>: pid <pid> <executable> cpu <cpu>` per marked process outside it.
A holder that runs a tier in process, as a Go-side gate caller does, can have a wide tree, so the line names at most `gateHoldProcessMax`, 20, descendants and 20 marked processes, each list ordered by CPU time, most first, and ends a cut list with `; and <n> more`, per the Added after Discussion entry "Gate-hold line caps listed processes".
A post that succeeds appends the key to `GateHolds`; one that fails logs through `gateHoldPosts` and is tried at the next probe tick.
Keys of holds no longer held are dropped from the entry, so a new hold of the run, with another PID or start, is noticed again.
The entry is created when absent, whatever the run's watch level, and a dobby restart keeps the keys in the state record.
The step reads `/proc` and the slot records only: it kills nothing, frees no slot and spawns nothing.

`doc.go` gains a "The gate-hold notice" section naming the bound, the awake clock, the one notice per hold and its content.

Tests: `TestGateHolds` is a table over `newPassHarness` probe ticks with one listed run `task` whose `TaskWorktree` is `/hub/task`, fake `Holders` (counting its calls), `Tree`, `MarkedPIDs` and a recording `Post`, and a `LoadConfig` answering the row's factor and minimum.
The fake `Tree` answers by PID: the holder's PID gives its tree, and each marked PID outside it gives its own one-entry sample. Rows:
- a tier3 holder under `/hub/task/sub` with `TierTimeout` 35 minutes and factor 1.5 posts nothing at 52 minutes of awake time and one notice at 53, naming every slot, the tier, the PID, each child PID of `Tree` and a marked process outside the tree, the exact line pinned once;
- a `suspend` inside the hold delays the notice past 53 minutes of wall time;
- the same hold on later probe ticks, and after `restartDobby`, posts no second notice;
- one PID holding two slots posts one notice from its earliest start;
- a new hold with another PID, and one with the same PID and a later start, posts again;
- a holder under `/hub/task-b` posts nothing;
- a holder with no tier takes `GateHoldMin`;
- factor 0 posts nothing;
- a failing `Holders` posts nothing, and the check posts once it reads again;
- a failing `LoadConfig` posts nothing and the check posts once it loads again; `startReady` makes `tick` return an error on a failed `LoadConfig` and `passHarness.tick` fails the test on any tick error, so this row calls `d.gateHolds` directly on a probe-due daemon rather than through `tick`;
- a non-probe tick calls no `Holders` and posts nothing;
- a failing `Tree` or `MarkedPIDs` still posts one notice that names the tree as unreadable;
- a failing `TaskWorktree` posts nothing for the run;
- a failing `Post` records no key, and the next probe tick posts the notice and records it;
- a holder whose tree holds more than 20 descendants names 20 and ends with `; and <n> more`;
- a run whose watch level reads off still gets its gate-hold notice.

**ImpactSummary:** New probe-tick step and seam struct inside `dobbyengine`; it writes only the `GateHolds` keys of card 3's watch entries, and nil seams keep every existing test unchanged.

**Commit:** 6: Notice a gate slot a driven run holds past its bound
