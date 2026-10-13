# Card 20 — batten-binary-restep

**Edit:**
- `internal/battenshed#InnerRunDeps`
- `internal/battenshed#innerRunProducer`
- `internal/battenshed#innerRunProducer.haltedStep`
- `internal/battenshed/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/battenshed#innerRunProducer.binaryRestep` -> `func (p *innerRunProducer) binaryRestep(ctx context.Context, w *childWait) *waitEnd`
- `plan:internal/battenshed#binaryRestepFile` -> `func binaryRestepFile(scratchDir, producer string) string`
- `plan:internal/battenshed#binaryRestepCeiling` -> `const binaryRestepCeiling = 30 * time.Minute`
- `plan:internal/battenshed#TestInnerRun_BinaryChangeRestepsADriverlessGoChild` -> `func TestInnerRun_BinaryChangeRestepsADriverlessGoChild(t *testing.T)`

**Uses:**
- `internal/battenshed#innerRunProducer.driverAtWork`

**Intent:**
D5's batten half: after a Go driver hands back and exits, a deploy of a new `lyx` re-steps the halted child once, through its bootstrap, so a fix lands without a parent turn.
A standalone go-driven run with no batten has no such watch (A7).

`InnerRunDeps` gains two seams, each resolved on Call:
- `ChildBuildChanged func() (build string, changed bool, err error)`: the label of the `lyx` on disk and whether it differs from the child's last step's recorded build;
- `ChildDirtyPaths func() ([]string, error)`: the task pair's uncommitted or untracked paths outside the run's records directories.
A nil `ChildBuildChanged` switches the re-step off.

`haltedStep` calls `binaryRestep` after the publish-retry step and the revive.
`binaryRestep` re-steps a halted child only when every condition below holds:
- the child is `blocked` or `failed`;
- `DriverStrand` reports `ChildDriverNone`, so an LLM-driven child, which always has a driver strand recorded, never qualifies;
- neither the child's run lock nor its driver lock is held, read through `driverAtWork`;
- the Call's publish-retry decision takes no action, so batten never re-steps a halt its publish retry is taking;
- `ChildBuildChanged` reports a changed build;
- `ChildDirtyPaths` reports none, the same clean-tree check the Go driver's hand-over makes;
- the re-step marker `binaryRestepFile(scratchDir, name)`, beside the publish-retry marker, does not already name this halt's stop key and this build.
It then calls `Spawn`, which runs the child's `lyx loom start`, whose go arm spawns `lyx loom drive`, and writes the marker (stop key and build label) only after `Spawn` returns nil.
That is once per halt episode and build, surviving a batten restart.
A refused spawn (`ErrChildNotParked`, `ErrChildDriverBusy`) writes no marker; it, or a dirty tree, is logged and retried at the next due check, never failing the row.
A batten crash between a spawn and the marker write is bounded by start's driver-lock check, which spawns no second driver.
A pause covering the slug holds it through `runArm`, as every halted-arm action is.

The check reads a build and a dirty-path list, so it spawns a process.
Per `PATTERN-daemon-wakeups` it runs at most once per due interval: the interval starts at `NoticeProbe`, doubles while the child's state and the on-disk build stay unchanged, is capped at `binaryRestepCeiling`, and returns to `NoticeProbe` on a change.
`innerRunProducer` carries the interval and the time of the last check.

`doc.go` describes the re-step, its conditions, its marker and its backoff.

Test `TestInnerRun_BinaryChangeRestepsADriverlessGoChild` in `innerrun_halted_test.go`, a table over the existing `newInnerRunDeps` fakes and fake clock, plus fake build, dirty-path and driver-lock seams.
Each Call ends through the scripted statuses: after the checks a row needs, `ReadStatus` returns the child `running`, which the wait records as a state change, so no row waits on a halted child forever.
- a driverless blocked child with a changed build and a clean tree spawns once and writes the marker, and a second check in the same episode spawns nothing;
- a fresh producer over the same scratch directory, as after a batten restart, spawns nothing for the same stop key and build;
- a newer build label spawns again;
- a `Spawn` refusing with `ErrChildNotParked` writes no marker, and the next due check spawns again and then writes the marker;
- a `failed` child qualifies the same way, and a `paused` one does not;
- a dead or live driver strand, a held run lock, a held driver lock, a publish retry taking the halt, a dirty tree, a tree whose only change is a new untracked file, and an unchanged build each spawn nothing;
- with nothing changing, the build seam is read at `NoticeProbe`, then at twice it, up to the ceiling.
`test-budget.yaml`'s `internal/battenshed` row rises by one.

**ImpactSummary:** one new action in batten's halted arm behind two optional seams that default it off; an LLM child, a pause, and an unchanged build keep today's halted behavior.
