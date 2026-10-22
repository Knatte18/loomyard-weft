# Card 2 — internal/runproc: the owner record and the one end sequence

**Create:**
- `internal/runproc/doc.go`
- `plan:internal/runproc#HubOwner` -> `const HubOwner = "_board"`
- `plan:internal/runproc#RoleDriver` -> `const RoleDriver = "driver"`
- `plan:internal/runproc#RoleLoop` -> `const RoleLoop = "loop"`
- `plan:internal/runproc#RoleBatten` -> `const RoleBatten = "batten"`
- `plan:internal/runproc#RolePush` -> `const RolePush = "push"`
- `plan:internal/runproc#RoleSync` -> `const RoleSync = "sync"`
- `plan:internal/runproc#Record` -> `type Record struct`
- `plan:internal/runproc#ChildRecord` -> `func ChildRecord(pid int, role string, argv []string) Record`
- `plan:internal/runproc#SelfRecord` -> `func SelfRecord(role string) Record`
- `plan:internal/runproc#Register` -> `func Register(dir, owner string, rec Record) error`
- `plan:internal/runproc#List` -> `func List(dir, owner string) ([]Record, error)`
- `plan:internal/runproc#Owners` -> `func Owners(dir string) (map[string][]Record, error)`
- `plan:internal/runproc#Ended` -> `type Ended struct`
- `plan:internal/runproc#SurvivorsError` -> `type SurvivorsError struct`
- `plan:internal/runproc#SurvivorsError.Error` -> `func (e *SurvivorsError) Error() string`
- `plan:internal/runproc#End` -> `func End(dir, owner string, callerPID int) ([]Ended, error)`
- `plan:internal/runproc#EndProcess` -> `func EndProcess(pid int, startTime, role string) ([]Ended, error)`
- `plan:internal/runproc#ender` -> `type ender struct`
- `plan:internal/runproc#ender.end` -> `func (e ender) end(dir, owner string, callerPID int) ([]Ended, error)`
- `plan:internal/runproc#ender.endProcess` -> `func (e ender) endProcess(pid int, startTime, role string) ([]Ended, error)`
- `plan:internal/runproc#lister` -> `type lister struct`
- `plan:internal/runproc#TestRegisterAndList` -> `func TestRegisterAndList(t *testing.T)`
- `plan:internal/runproc#TestEnd` -> `func TestEnd(t *testing.T)`
- `plan:internal/runproc#TestLeafInvariant_AllowlistOnly` -> `func TestLeafInvariant_AllowlistOnly(t *testing.T)`
- `pattern/PATTERN-run-process-owner.md`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/proc#ListStats`
- `plan:internal/proc#SignalProcess`
- `plan:internal/proc#SignalGroup`
- `internal/testkit/scankit#AssertImportAllowlist`

**Intent:**
Add the owner record of D1 and the end sequence of D3 as a new leaf package, importing only the standard library, `internal/proc`, `internal/state` and `internal/logger` (the declared `PATTERN-leaf-packages` change).
The package is told its directory and derives no path (`PATTERN-told-geometry`); card 3 declares the directory.

Record (D1).
`Record` holds `PID`, `PGID` (the pid when the process leads its own group, zero otherwise), `StartTime`, `Role`, `Argv` and `RegisteredAt`, with snake_case JSON tags.
`ChildRecord` builds one for a child a spawner just started: start time from `proc.StartTime`, `PGID` equal to the pid on every platform but Windows, since `proc.Detach` starts the child with `Setsid` on Linux and a Windows child's group cannot be signalled.
`SelfRecord` builds one for the calling process from `os.Getpid()`, `os.Args` and its own group, read from `ListStats` (`PGID` set only when the group id equals the pid), so a self-registered role started inside a shell's group is never group-signalled.
`Register` writes `<dir>/<owner>/<pid>-<start>.json` through `state.WriteJSON` (atomic, unknown keys kept, `PATTERN-tolerant-records`), replacing a file of the same pid and start time, and logs the registration at Info (`PATTERN-spawn-observability`).
`HubOwner` is `_board`, a name `fabricengine`'s slug check refuses as reserved hub geometry, so no slug can take it; its doc comment says so.
`List` returns the live records of one owner, reading each file tolerantly (a file that does not decode is skipped), and removes every file whose pid is dead or whose start time differs from `proc.StartTime`'s; no process removes its own file.
On Windows, where no start time can be read, a record whose `StartTime` is empty is kept while `proc.IsAlive` reports the pid alive.
`List` reads liveness through `lister{startTime, isAlive}`, the production seams `proc.StartTime` and `proc.IsAlive`, so the test reaches the Windows branch on Linux.
`Owners` returns `List` for every owner directory.

End (D3).
`End` is `ender{}.end` with the production seams and engine constants: `endGrace` five seconds in all after SIGTERM, `killBound` two seconds after SIGKILL, `endPasses` three.
`ender` carries the seams the test drives: `stats` (`proc.ListStats`), `startTime` (`proc.StartTime`), `isAncestor` (`proc.IsAncestor`), `signalProcess`, `signalGroup`, `sleep`, `now` and `goos`.
One pass: list the owner's live records; leave out the caller pid and every ancestor of it; with one `stats` reading, collect for every record at once its descendants by parent pid with their start times; then SIGTERM each leader's group (only while a live entry reports that group id with a start time at or after the record's, as `proc.KillTree` checks), each non-leader by pid (only after its start time still matches), and each read descendant outside a group already signalled, through its own group when the descendant leads one;
poll `stats` until every signalled pid is gone or reads `Zombie` (a zombie counts as gone), bounded by `endGrace`; SIGKILL what is left the same way; poll again up to `killBound`.
After a pass `end` re-lists the owner and repeats on any live record it has not yet ended, at most `endPasses` passes, so a child registered during the end is reached.
It returns each process it ended as `Ended{PID, Role}` (a descendant takes its record's role with a `descendant of` prefix), and, when any process survives the last poll, a `*SurvivorsError` whose `Survivors` lists each survivor's pid and role and whose message names them.
Every signal and the outcome are logged at Info.
With `goos` windows, or when `stats` returns `errors.ErrUnsupported`, `end` signals nothing and returns no error.
`EndProcess` is `ender{}.endProcess` over the same seams: it runs the same SIGTERM, grace, SIGKILL, bound sequence on one pid of an unregistered process after its start time still matches, for `lyx tender reap`, and returns a `*SurvivorsError` for a process alive after `killBound`.

Tests.
`TestRegisterAndList` (untagged, against the test's own pid and a temp dir under `internal/lyxtemp`): a registered self record lists back with its fields; a second `Register` with the same pid and start time leaves one file; a record whose pid is dead, and one whose start time differs, are not listed and their files are gone after the read; a file holding an undecodable body and one holding an unknown key are respectively skipped and read; `Owners` groups by owner;
`SelfRecord`'s `PGID` is the pid when `ListStats` reads the test process as its group's leader and zero otherwise;
through `lister` with a `startTime` that reads nothing, a record with an empty `StartTime` is kept while `isAlive` reports it alive and pruned once it does not.
`TestEnd` drives `ender.end` over fake seams (a scripted process table, a fake clock whose `sleep` advances `now`, a signal recorder), one row per D3 behaviour, so no row spawns or waits: a recorded leader ends with one group signal; a non-leader ends with its descendants signalled by pid; a descendant leading its own group (a push's remote `git` child) gets a group signal; the caller and an ancestor of it are not signalled; a recorded pid whose start time differs is left alone; a record that appears during the first pass is ended by the second; a process that ignores SIGTERM is killed after `endGrace` on the fake clock; a process that survives SIGKILL yields a `*SurvivorsError` naming its pid and role once `killBound` is spent; a process that reads `Zombie` after SIGTERM counts as ended with no SIGKILL; `goos` windows signals nothing, and so does a `stats` seam returning `errors.ErrUnsupported`;
`endProcess` leaves a pid whose start time differs alone, ends a matching one by SIGTERM, kills one that ignores SIGTERM after `endGrace`, and returns a `*SurvivorsError` for one that survives SIGKILL.
`TestLeafInvariant_AllowlistOnly` pins the import set with `scankit.AssertImportAllowlist`.

Docs and PATTERN.
`doc.go` states the record, its directory being told, pruning on read, `HubOwner`, the end sequence, its constants and its Windows bound.
`PATTERN.md` gains, under `## Daemons and waits`, the declared entry:

```markdown
- `PATTERN-run-process-owner` — Starting a detached or long-lived process for a run: register it in `internal/runproc`'s owner record; ending a run's processes: only via `runproc.End`. — [background](pattern/PATTERN-run-process-owner.md)
```

`pattern/PATTERN-run-process-owner.md` states who registers under which owner (D2), that `End` alone ends a run's processes and that `EndProcess` serves only reap's unregistered orphans.
`pattern/PATTERN-leaf-packages.md` gains a `## internal/runproc` section: imports only the standard library, `internal/proc`, `internal/state` and `internal/logger`.
`docs/overview.md` gains a module-table row for `internal/runproc/` between its neighbours.

**ImpactSummary:** new leaf package plus PATTERN and overview rows; no existing code changes.

**Commit:** 2: internal/runproc: one owner record per run process, and runproc.End
