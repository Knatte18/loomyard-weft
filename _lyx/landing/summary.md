# Gate test names a tier's -trimpath GOFLAGS beside its summary line

A hub's tiers set `-trimpath` in their GOFLAGS on purpose, so a fresh worktree reuses the build and test cache.
Under it `runtime.Caller` reports module-relative file paths, so a test that locates its repository from one fails with "not an absolute path" under `lyx gate test`, while a raw `go test` passes.
Nothing in the gate's output said so, and agents fell back to raw `go test`.

The gate keeps `-trimpath` and now surfaces it.
When a tier's effective GOFLAGS carries an enabled `-trimpath`, `lyx gate test` prints one line to stderr just before the tier's summary line and writes the same line at the top of the tier's run log.
The line names the tier, its GOFLAGS value and the way forward: locate the repository from the working directory or `go.mod`, never from a `runtime.Caller` path alone.
`internal/gatecli/doc.go` documents the line under a new `# Trimpath` section.

Closes #807
Closes #808

## Design decisions

- **The child's view decides.**
  `trimpathLine` reads the last `GOFLAGS` entry of the environment the tier's commands get: the gate's inherited environment plus the tier's own `env`.
  A GOFLAGS inherited from an outer gate run therefore counts as well as one the tier sets.
  Within that value the last `trimpath` field decides, and its bool value is parsed as go parses it, so `-trimpath`, `--trimpath`, `-trimpath=true` and `-trimpath=1` fire, and `-trimpath=false` or `-trimpath -trimpath=false` stay silent.
- **Once per tier run that has an invocation.**
  A tier with an empty scope runs no build and gets no line.
  The `--guards` step goes through the same tier run, so it prints the line too.
- **Beside the summary line, not before the first command.**
  `report` prints the line, so a refused tier still prints only its JSON envelope on stdout.
  The log gets the line before the first command's output.
- **No run-record field.**
  The record's JSON is unchanged; the log carries the line.
- **Not covered.**
  A `-trimpath` given on a tier's command line rather than in GOFLAGS gets no line; it shows under `-vv` and in the run record.
- `-trimpath` is not removed from any GOFLAGS, the hub's `gate.yaml` or the non-tier slot holders' `gateslot.HolderEnv`.

## To check by hand

Review ended without converging, and these were its findings; each was addressed by a later commit, but a reader should confirm:

- `internal/gatecli/tierrun.go:57`: a tier with an empty scope and a `-trimpath` GOFLAGS prints no line; the empty-scope subtest in `internal/gatecli/scopes_integration_test.go` should assert it.
- `internal/gatecli/tiers.go:268`: `GOFLAGS=-trimpath -trimpath=false` gets no line, as go keeps the last occurrence of a repeated flag.
- `internal/gatecli/tiers.go:270`: other enabling spellings such as `-trimpath=1` fire the line.
- `internal/gatecli/test_integration_test.go:1160`: the `--guards` test should pin that the guard step prints the line once per run that reached the report.
