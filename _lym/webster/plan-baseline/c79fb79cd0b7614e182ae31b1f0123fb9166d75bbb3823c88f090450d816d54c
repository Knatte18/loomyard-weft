# Card 3 — driver-wait-readers

**Create:**
- `plan:internal/sheddriver#ReadDriveWait` -> `func ReadDriveWait(recordPath, driverLockPath string) (wait DriveWait, standing, driverGone bool, err error)`
- `plan:internal/sheddriver#TestReadDriveWait` -> `func TestReadDriveWait(t *testing.T)`
- `plan:internal/loomcli#TestLoomStatusExtras` -> `func TestLoomStatusExtras(t *testing.T)`

**Edit:**
- `internal/loomcli#loomCLI.failedHalt`
- `internal/loomcli#loomCLI.loomStatusExtras`
- `internal/loomcli#TestLoomPostRun_HaltNotes`
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#daemon.watchRun`
- `internal/dobbyengine#watchStop`
- `internal/dobbyengine#WatchReading`
- `internal/dobbyengine#TestRunWatch`
- `internal/battencli#DobbyRuns`
- `internal/battencli#TestDobbyRuns_ReadsTheKeptCancelTheHandOverMarkTheOwnerAndTheBoltHold`
- `internal/dobbyengine/doc.go`

**Uses:**
- `internal/sheddriver/doc.go`
- `internal/loomcli/doc.go`
- `internal/sheddriver#DriveWait`
- `internal/shedrun#DriveWaitRecord`

**Intent:** D6: a run the Go driver waits on reads as waiting, not as a stop, and mails nothing.

`ReadDriveWait` reads the `DriveWait` record at `recordPath` and whether a process holds the driver lock at `driverLockPath`, by a non-blocking read-lock try it releases at once.
`standing` is true while the record exists; `driverGone` is true when the record stands and the lock is free, a driver that died.
A missing record reads as not standing; a failed read returns its error.

A step run under the Go driver writes no halt note for a failed stop whose transient class is `git-transport` or `github-api`:
`loomCLI.failedHalt` reports no halt for such a status while the driver lock of the run is held, since that driver waits on the stop.
A step run outside the Go driver, with the lock free, keeps today's note.
The note of a waited run comes from the step whose stop is no longer network-class, which `failedHalt` treats as today.
`loomStatusExtras` adds a `waiting` key, the record's producer, class and start time, while `ReadDriveWait` reads it standing under a held lock.

`RunView` gains `Waiting`, `WaitDied`, `WaitProducer`, `WaitClass` and `WaitSince`, which `DobbyRuns` fills for a pair that stands here,
and for every run also from the batten run's own record and driver lock in the prime, `shedrun.DriveWaitRecord` and `shedrun.DriverLock` over the prime and the slug, so a batten run waiting at Land reads `Waiting` too.
`Waiting` holds while `ReadDriveWait` reads the record standing under a held lock,
and also while the driver lock is held and the child's status is failed with a `git-transport` or `github-api` class, the gap between the step's failed status and the driver's first record write, so no tick in that gap reads the run as stopped.
The run watch's `watchRun` keeps a `Waiting` run's last reading and owes no line for it, so it mails nothing while the driver waits.
A `WaitDied` view reads as a stop once: `WatchReading` gains `WaitDied`, `watchStop` reads it as a stop, and the line names the producer and class the record holds, as the stop it is.
D3's summary (card 20) reads its waits through `ReadDriveWait` over every run of the hub, not through `RunView`.
`internal/dobbyengine/doc.go`'s run-watch section says that a `Waiting` run owes no line, and that a `WaitDied` run reads as one stop naming the producer and class its record holds.

Tests, tier0, adding about a second to one tier0 run.
`TestReadDriveWait` pins no record, a record under a held lock, a record with the lock free, and a record that does not decode returning its error.
`TestLoomPostRun_HaltNotes` gains rows: a `git-transport` and a `github-api` failed status with the driver lock held write no note, and a `git-transport` one with the lock free writes the transient-stop note.
`TestLoomStatusExtras` pins the `waiting` key under a held lock and its absence with no record.
`TestRunWatch` gains rows: a run listed `Waiting` over several ticks owes no line, and the same run then listed `WaitDied` owes exactly one stop line naming the producer.
`TestDobbyRuns_ReadsTheKeptCancelTheHandOverMarkTheOwnerAndTheBoltHold`, tier1, gains rows: a batten run whose wait record stands in the prime under its held driver lock reads `Waiting` with producer `Land`, and with the lock free reads `WaitDied`.

**ImpactSummary:** Adds fields to the dobby's run view and two loom reads; the halt note's rule narrows for network stops under a live driver only.
