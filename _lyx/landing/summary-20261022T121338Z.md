# Gate test names a tier's -trimpath GOFLAGS beside its summary line

A hub's gate tiers set `-trimpath` in their GOFLAGS on purpose, so a fresh worktree reuses the build and test cache.
Under `-trimpath`, `runtime.Caller` reports module-relative file paths.
A test that locates its repository from one fails under `lyx gate test` while a raw `go test` passes, and nothing in the gate's output said why (issues #807 and #808).
`-trimpath` stays, as the operator decided, and the gate now says when a tier builds with it.

## What changes

- `trimpathLine` in `internal/gatecli/tiers.go` reads the last `GOFLAGS` entry of a tier's environment, the inherited one or the tier's own, since that is the value the child sees.
  It fires on a `-trimpath`, `--trimpath` or `-trimpath=true` field, and not on `-trimpath=false`.
  The line names the tier, the GOFLAGS value, the effect on `runtime.Caller` and the way forward: locate the repository from the working directory or `go.mod`.
- `runTier` writes the line at the top of the tier's run log, before the first command's output, and hands it back through `tierResult`.
- `report` prints it to stderr just before the tier's summary line.
  A refused tier never reaches that print, so its stdout stays one JSON envelope.
  The guard step goes through the same path and gets the line too.
- A tier with an empty scope runs no command and gets no line.
- `internal/gatecli/doc.go` gains a `# Trimpath` section that documents why `-trimpath` stays and what the line says.

## Design decisions

- The line goes to stderr just before the summary line, not before the first command as the decision record states.
  Printed first, it came ahead of a refused tier's JSON envelope on stdout and broke callers that parse that envelope.
  Next to the summary line, it also sits beside a failing run's verdict, where a reader looks.
- The run record's JSON gains no field, because the log already carries the line.
- A `-trimpath` passed on a tier's command line instead of in GOFLAGS gets no line, since the command already shows in the run record and under `-vv`.

## Tests

- `TestTrimpathLine` is a table over the detection forms: plain, double-dash and `=true` fire, `=false` and an absent flag stay silent, and the last GOFLAGS entry wins.
- The `TestGateTest_Scenario` subtest whose tier sets `GOFLAGS: -trimpath -p={parallel}` now checks that the line is in the gate's output and that the run log holds exactly the line, ahead of the stand-in's empty output.
- `lyx gate test --upto 1` passes over the computed scope and its guard step.
