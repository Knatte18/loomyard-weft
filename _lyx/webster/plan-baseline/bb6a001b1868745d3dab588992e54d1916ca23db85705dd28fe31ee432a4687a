# Card 1 — gateslot-pool

**Create:**
- `internal/gateslot#Pool`
- `internal/gateslot#Limits`
- `internal/gateslot#Holder`
- `internal/gateslot#Lease`
- `internal/gateslot#Pool.Acquire`
- `internal/gateslot#Pool.Holders`
- `internal/gateslot#Pool.Inherited`
- `internal/gateslot#Lease.Release`
- `internal/gateslot#Lease.Env`
- `internal/gateslot#InheritEnv`
- `internal/gateslot#GoFlags`
- `internal/gateslot#Dir`
- `internal/gateslot#Wait`
- `internal/gateslot#WaitDir`
- `internal/gateslot#WriteWait`
- `internal/gateslot#ReadWaits`
- `internal/gateslot/doc.go`
- `internal/gateslot/pool_test.go`
- `internal/gateslot/pool_integration_test.go`
- `internal/gateslot/testmain_test.go`

**Edit:**
- `cmd/lyx/pathrules_test.go`
- `cmd/lyx/testdata/test-budget.yaml`
- `docs/overview.md`

**Uses:**
- `internal/lock#TryAcquireWriteLock`
- `internal/lyxdirs#DotLyxDirName`
- `internal/logger#Info`

**Intent:**
Create `internal/gateslot`, the engine that owns the hub-wide pool of gate slots (D4).
It is told everything it works on and derives no path of its own beyond the `Dir`/`WaitDir` joins, which spell `.lyx` only through `lyxdirs` (`PATTERN-told-geometry`, `PATTERN-lyxdirs-single-declarer`).

- `Pool` holds the told slot directory (`Dir`), a `Limits func() (Limits, error)` read at every acquire so a lowered count takes effect on the next acquire while a running holder of a higher slot finishes, and an injectable `Poll` interval (a zero value means a production default of a couple of seconds).
- `Limits` carries `Slots`, `GoParallel` (the `-p` cap) and `CLIWait` (a `time.Duration`).
- Each slot is one OS file lock through `internal/lock` at `<Dir>/slot-<n>.lock`, `n` from 1 to `Slots`.
  `Acquire` tries every slot with `TryAcquireWriteLock`; on success it writes the holder record (`Holder`: worktree, site or command, pid, start time) as YAML beside the lock and returns a `Lease`.
  When every slot is held it logs the wait start through `internal/logger`, sleeps `Poll` (cancellable by `ctx`) and retries, and logs the wait end when it acquires; a cancelled `ctx` returns its error with no slot held.
  It imposes no deadline of its own: a caller wanting a bound passes a deadline context.
  Polling gives no FIFO order; that bound is D4's.
- `Lease.Release` removes the holder record and releases the lock; an OS lock is released by process death too, so a crashed holder never leaks a slot.
- `Lease.Env(base)` returns a copy of `base` with `GOFLAGS` set to `GoFlags(<base's GOFLAGS>, GoParallel)` and `InheritEnv` set to the held slot's lock path.
  `GoFlags` appends `-p=<cap>` to an existing value, space-separated, or returns `-p=<cap>` alone.
- `Holders` returns the holder records of the slots whose lock is held right now, trusting a record only while its lock is held (a lock it can take is free and is released at once); it reads every `slot-*.yaml` in the directory, so a holder of a slot above a lowered count still shows.
- `Inherited(slot)` reports whether `slot` names a lock file inside `Dir` whose lock is held, the check `lyx gate test` runs before running inside a parent's slot without acquiring.
- `Dir(boardDir)` is `<boardDir>/.lyx/gate`, the hub-wide slot directory; `WaitDir(anchorRoot)` is `<anchorRoot>/.lyx/gate`, where a waiter with no `verifytree` marker (`lyx gate test`, the card-verify rerun, the plan gate's type load) keeps its per-worktree wait record.
- `Wait` (site — the `lyx gate test` command or the Go-side site label — pid, start time) is that wait record: `WriteWait` writes one uniquely named YAML file into `dir` and returns its path, which the writer removes; `ReadWaits` returns the records in `dir` whose pid is alive (`proc.IsAlive`), so a crashed waiter never shows.

`doc.go` states the package's role, the slot and holder-record layout, the inheritance variable and the bounds above.
Tests: `pool_test.go` is tier 1 (file locks in a temp dir are not spawns, poll interval shortened), table-driven over acquire with free slots, a second acquirer waiting under `Slots: 1` while the first holds and proceeding once it releases, the wait start and end log lines, a cancelled wait holding no slot, `Holders` naming the live holder and dropping a record whose lock is free, `Inherited` true for a held slot and false for a free one or a path outside `Dir`, `Env` composing `GOFLAGS` with and without an existing value, `ReadWaits` dropping a dead pid's record.
`pool_integration_test.go` pins that a holder process that exits without releasing frees its slot, by re-executing the test binary as a child that acquires and is killed; `testmain_test.go` runs the package through `tmuxkit.Main` (`PATTERN-test-isolation`).
`cmd/lyx/pathrules_test.go` gains rows for `Dir` (hub class, over the location's board dir) and `WaitDir` (transient class).
`test-budget.yaml` gains the `internal/gateslot` row.
`docs/overview.md`'s module tree gains the `internal/gateslot/` line.

**ImpactSummary:** New engine package with no callers yet; the only existing files touched are the path-rule table, the test budget and the overview's module tree.
