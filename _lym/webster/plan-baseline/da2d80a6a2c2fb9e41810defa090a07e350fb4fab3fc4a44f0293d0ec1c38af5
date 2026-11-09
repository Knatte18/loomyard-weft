# Card 3 — adopt-record

**Create:**
- `internal/adoptrecord/doc.go`
- `internal/adoptrecord/record.go`
- `internal/adoptrecord/record_test.go`
- `internal/adoptrecord/testmain_test.go`
- `internal/adoptrecord#FileName`
- `internal/adoptrecord#LockName`
- `internal/adoptrecord#StepReconcile`
- `internal/adoptrecord#StepPairs`
- `internal/adoptrecord#StepPause`
- `internal/adoptrecord#StepOwner`
- `internal/adoptrecord#StepDeploy`
- `internal/adoptrecord#StepBeam`
- `internal/adoptrecord#StepCheck`
- `internal/adoptrecord#StepDone`
- `internal/adoptrecord#Steps`
- `internal/adoptrecord#StateRunning`
- `internal/adoptrecord#StateStopped`
- `internal/adoptrecord#StateDone`
- `internal/adoptrecord#StateInterrupted`
- `internal/adoptrecord#Request`
- `internal/adoptrecord#Request.SameSubjects`
- `internal/adoptrecord#Request.Command`
- `internal/adoptrecord#StepRecord`
- `internal/adoptrecord#Inputs`
- `internal/adoptrecord#Record`
- `internal/adoptrecord#Record.Done`
- `internal/adoptrecord#Record.ResumeStep`
- `internal/adoptrecord#Record.ReadState`
- `internal/adoptrecord#Record.Unfinished`
- `internal/adoptrecord#Path`
- `internal/adoptrecord#LockPath`
- `internal/adoptrecord#Read`
- `internal/adoptrecord#Write`
- `internal/adoptrecord#Remove`
- `internal/adoptrecord#WorkerAlive`
- `internal/adoptrecord#ErrDecode`
- `internal/adoptrecord_test#TestRead`
- `internal/adoptrecord_test#TestRecordReadState`
- `internal/adoptrecord_test#TestRecordResumeStep`
- `internal/adoptrecord_test#TestRequestSameSubjects`
- `internal/adoptrecord_test#TestRequestCommand`

**Edit:**
- `internal/fabricengine#AdoptScratchDir`
- `internal/fabricengine/doc.go`
- `internal/lymdirs#AdoptDirName`

**Uses:**
- `internal/lymdirs/doc.go`

**Intent:**
D2's adopt record as its own package, `internal/adoptrecord`, so `beamcli` reads it without the adopt module's code.
`doc.go` documents the package: the record, its path, its lock and its lock-free read.

The file is `adopt.json` directly under `fabricengine.AdoptScratchDir(hubPath)`, beside the subject directories, and `adopt.lock` beside it (`PATTERN-lym-files`: never-tracked, under the hub's scratch directory).
`Path` and `LockPath` join those names.
`AdoptScratchDir`'s comment and the fabricengine `doc.go` sentence on it both say the directory also holds the adopt record, its lock and the worker's output.
`lymdirs.AdoptDirName`, the one declarer of that directory's name, says the same:
its comment names the adopt record, its lock and the worker's output beside the check files.

`Request` holds the request: `Slugs`, `Root`, `All`, `KeepLocal`, `DropLocal`, `NoOrch`.
- `SameSubjects` reports whether two requests name the same subjects: two `--all` requests match, whatever set each resolves to; otherwise the same slug set, order-free, and the same `Root`.
  A `--all` request never matches a named one.
- `Command` renders the `lym adopt` command line of the request: its slugs sorted, `root`, `--all`, and each set flag in a fixed order.
  Every refusal and way forward that names the re-run of the same `lym adopt` uses it.

`StepRecord` is one step that ended: `Name`, `EndedAt`, `Outcome` (the part of the end envelope that step contributes, kept as `json.RawMessage`).

`Inputs` holds what the later steps need:
`LocalRuns` and `Moved` (the runs before and after the reconcile);
`Runs` and `Adoptable`;
`RootGoing`;
`HeldBack` (the projects a local-commit stop held back);
`OwnerStood` (the beams whose owner write stood);
`PrimeRecorded`;
`Paused` (the slugs the `pause` step paused, converted beam slugs included).

`Record` holds the `Request`, the resolved `Subjects` (the beam slugs, `root` when root goes), the worker's `PID`, `StartTicks` and `Start`;
the `Steps` done with their end times and outcomes, the step in flight (`InFlight`), the `Inputs` and the `State`;
for a stop, `Error`, `WayForward` and `StoppedPart`, the part the stopped step gathered before its error (`json.RawMessage`);
at `done` or a stop, the `Envelope` the worker writes (`map[string]any`).
Every field carries a JSON tag; unknown keys are tolerated (`PATTERN-tolerant-records`).

- `Done(step)` reports whether `Steps` lists the step.
- `ResumeStep` returns the first step of `Steps` not done.
  When that step is `owner` or earlier, it returns `reconcile` instead, so a resumed owner write always follows a fresh reconcile.
- `ReadState` returns `State`, except `interrupted` for a `running` record whose worker `alive` reads gone: a pid not alive, or alive with other start ticks.
- `Unfinished` reports whether the state is anything but `done`.
- `WorkerAlive` is the production `alive`: `proc.IsRunning`, so an exited worker its parent has not reaped yet reads gone, and, when both sides read start ticks, equal ticks; a record without ticks reads alive on the pid alone.

`Read` stats the file first: an absent record reads false and creates no directory or lock file.
It reads the file without a lock, since `Write` replaces it atomically, and decodes it tolerating unknown keys.
A record that does not decode is an error wrapping `ErrDecode`, naming the path and `lym adopt --discard`;
`ErrDecode` lets `--discard` tell it from an I/O error and remove such a file (card 8).
`Write` writes through `state.WriteJSON` under the path's state lock, creating the directory.
`Remove` deletes the file; an absent file is no error.
None of them takes `adopt.lock`: the worker and `--discard` take it in `adoptcli`.

Tests, all tier0, in `internal/adoptrecord/record_test.go` (tier0), each table-driven over `t.TempDir()` or literal records:
- `TestRead`: an absent record reads false and leaves the scratch directory absent; a written record reads back equal; an unknown key still reads; an undecodable file is an error that `errors.Is` reads as `ErrDecode` and that names `lym adopt --discard`.
- `TestRecordReadState`: running with a live worker reads running; running with a dead pid, and with other ticks, reads interrupted; stopped and done read as written.
- `TestRecordResumeStep`: nothing done, and `reconcile`, `pairs` or `pause` done, resume at `reconcile`; `owner` done resumes at `deploy`; `deploy` done resumes at `beam`.
- `TestRequestSameSubjects`: `--all` against `--all` matches; the same slugs in another order match; another slug, a missing `root`, and `--all` against named slugs do not.
- `TestRequestCommand`: slugs, `root`, `--all` and each flag render in their order.

`internal/adoptrecord/testmain_test.go` (untagged) runs the package's tests under `tmuxkit.Main` with the hermetic git environment, as every package's test main does (`PATTERN-test-isolation`).

**ImpactSummary:** one new package read by adoptcli and beamcli; in fabricengine only the AdoptScratchDir comment and one doc sentence change, and in lymdirs only the AdoptDirName comment, no code.
