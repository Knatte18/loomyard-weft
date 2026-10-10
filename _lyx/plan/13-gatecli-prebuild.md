# Card 13 — gatecli-prebuild

**Create:**
- `plan:internal/gatecli#prebuildLyx` -> `func prebuildLyx(ctx context.Context, goBinary, root string) (bin string, cleanup func(), err error)`
- `plan:internal/gatecli#buildRoot` -> `func buildRoot(location *lyxcwd.Location, dir string) string`
- `plan:internal/gatecli#stripPrebuilt` -> `func stripPrebuilt(env []string) []string`
- `plan:cmd/testtiming#prebuildLyx` -> `func prebuildLyx(tags string) (env []string, cleanup func(), err error)`

**Edit:**
- `internal/gatecli#runTest`
- `cmd/testtiming#run`
- `cmd/testtiming#runRedundancy`
- `plan:cmd/testtiming#runResources`
- `internal/gatecli/test_integration_test.go`

**Uses:**
- `plan:internal/gateslot#PrebuiltLyxEnv`

**Prosa:**
- `internal/gatecli/doc.go`
- `docs/benchmarks/running-tests.md`

**Intent:** D7's gate half: `lyx gate test` builds `lyx` once for a tagged run and the harness does the same.
In `runTest`, after the slot is held and before `go test`: with a non-empty `--tags`, `prebuildLyx` runs `<goBinary> build -C <root> -o <tmp>/lyx ./cmd/lyx` inside the slot, with `<tmp>` a per-invocation `os.MkdirTemp`, logs the spawn and its end through `logger`, and the child environment gains `PrebuiltLyxEnv=<tmp>/lyx`; the cleanup removes `<tmp>` after `go test` ends.
`buildRoot` is the worktree root inside a hub and `dir` outside one; when `<root>/cmd/lyx` does not exist the build is skipped and nothing is exported, so a gate run inside a nested module or another repository is unaffected.
With no tags, `stripPrebuilt` removes an inherited value from the child environment, so an untagged run never sees one.
A build failure is reported on the envelope with the build's output and the exit code 1, before `go test` runs.
`cmd/testtiming`'s `prebuildLyx` does the same for its own tagged runs, with `go` from PATH and the module root, and `run`, `runRedundancy` and `runResources` call it: the environment it returns reaches each `go test` child, and the cleanup runs after the last child.
Tests: `TestGateTest_Scenario` gains two steps; the stand-in appends one record per invocation (a line of JSON) so a step can read both the build and the test invocation, and `standInRecord` gains a `Prebuilt` field.
Step one, tagged: the stand-in sees a `build -C <prime> -o <path> ./cmd/lyx` invocation before the `test` invocation whose environment carries the variable set to `<path>`, and `<path>`'s directory is gone once the gate exits.
Step two, untagged with the variable set in the gate's own environment: the stand-in sees no build and no variable.
The gatecli doc's `Verb` and `Inheritance` sections describe the build, the variable and the strip, and `running-tests.md` names the variable and that `lyx gate test --tags` is where one build serves every package.

**ImpactSummary:** `runTest` gains one spawn per tagged run; three harness entry points gain an environment and a cleanup; the child's `go test` arguments are unchanged.
