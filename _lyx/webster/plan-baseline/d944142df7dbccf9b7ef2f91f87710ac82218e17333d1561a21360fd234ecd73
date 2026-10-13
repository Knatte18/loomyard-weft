# Card 4 — on-disk-build-and-verb-bodies

**Create:**
- `plan:internal/buildvcs#OfFile` -> `func OfFile(path string) (Identity, error)`
- `plan:internal/buildvcs#TestOfFile` -> `func TestOfFile(t *testing.T)`
- `plan:internal/shedverbs#RunUntilStop` -> `func RunUntilStop(ctx context.Context, spec *Spec, argv []string, attempt string, out io.Writer) int`
- `plan:internal/shedverbs#RunGoto` -> `func RunGoto(ctx context.Context, spec *Spec, target string) (shedengine.Status, error)`
- `plan:internal/shedverbs#LastStepBuildChanged` -> `func LastStepBuildChanged(stepsDir string, against buildvcs.Identity) bool`

**Edit:**
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#gotoCmd`
- `internal/shedverbs#TestLastStepOf_BuildIdentity`
- `internal/shedverbs/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
The two reads and two bodies the Go driver calls in-process, so `lyx loom drive` restates no generic verb body (`PATTERN-shed-verb-set`) and judges a binary change by the build on disk (D5).

`buildvcs.OfFile` reads the VCS stamp of the Go binary at `path` through `debug/buildinfo.ReadFile` and maps its settings with `identityOf`, returning the read error for a file that is not a Go binary.

`shedverbs.LastStepBuildChanged` returns `lastStepOf(stepsDir, against).BinaryChanged`, false when there is no step record: true when the last step's recorded build is a known build other than `against`.
The Go driver and batten pass the identity `buildvcs.OfFile` read from the `lyx` on disk, so any difference counts, a rollback included, and the envelope's `binary_changed`, which compares against the calling binary, is never read for this.

`RunUntilStop` is the body of `step --until-stop` after its flags are parsed, moved out of `stepCmd` unchanged: the "this recipe arms no loop" refusal, then, for a non-empty `attempt`, the `Hooks.ConsumeAttempt` call with its nil-hook, busy and error refusals carrying `arbiter_attempt`, then `runUntilStop`.
It returns the exit code and prints the one envelope on `out`, exactly as the cobra body does today.
`stepCmd`'s `untilStop` branch keeps the `--loop-detached` branch and the attempt-without-until-stop refusal, and calls `RunUntilStop` for the waiter.

`RunGoto` is `gotoCmd`'s body after the target is known: the `PreGoto` hook, the status-lock directory, and `shedengine.Goto` over the spec's paths, returning the moved status or the error.
`gotoCmd` keeps the empty-target refusal and maps a `shedengine.ErrShedBusy` error to the busy-kind envelope as today.

`doc.go` says `RunUntilStop` and `RunGoto` are the exported entries an in-process driver calls, the same bodies the verbs run.

Tests:
- `TestOfFile` reads `os.Executable()`, the test binary, with no error, and refuses a temp file holding text.
- `TestLastStepOf_BuildIdentity` asserts `LastStepBuildChanged` is false for the recorded identity, true for a different known one, false for the legacy record and false for an empty directory.
The extracted bodies keep their behavior, which `TestLoop_ArbiterAttemptConsumedByTheWaiter`, `TestLoop_RunsUntilStopAndPrintsOneEnvelope` and the `TestGotoCmd_*` tests already pin through the verbs.
`test-budget.yaml`'s `internal/buildvcs` row rises by one.

**ImpactSummary:** two cobra bodies in `internal/shedverbs` split into exported functions with unchanged behavior, and two new pure reads; no caller outside the package changes.
