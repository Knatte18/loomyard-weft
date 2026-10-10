# Card 13 — gatecli-prebuild

**Create:**
- `plan:internal/gatecli#prebuildLyx` -> `func prebuildLyx(ctx context.Context, goBinary, root string) (bin string, cleanup func(), err error)`
- `plan:internal/gatecli#buildRoot` -> `func buildRoot(location *lyxcwd.Location, dir string) string`
- `plan:internal/gateslot#StripPrebuilt` -> `func StripPrebuilt(env []string) []string`
- `plan:cmd/testtiming#prebuildLyx` -> `func prebuildLyx(tags string) (env []string, cleanup func(), err error)`

**Edit:**
- `internal/gatecli#runTest`
- `cmd/testtiming#run`
- `cmd/testtiming#runRedundancy`
- `plan:cmd/testtiming#runResources`
- `internal/gatecli/test_integration_test.go`
- `internal/verifytree#Verify`
- `internal/verifytree/verify_integration_test.go`
- `internal/websterengine#runCardVerify`
- `internal/websterengine/cardverify_test.go`

**Uses:**
- `plan:internal/gateslot#PrebuiltLyxEnv`

**Prosa:**
- `internal/gatecli/doc.go`
- `internal/verifytree/doc.go`
- `internal/websterengine/doc.go`
- `docs/benchmarks/running-tests.md`

**Intent:** D7's gate half: `lyx gate test` builds `lyx` once for a tagged run and the harness does the same.
In `runTest`, after the slot is held and before `go test`: with a non-empty `--tags`, `prebuildLyx` runs `<goBinary> build -C <root> -o <tmp>/lyx ./cmd/lyx` inside the slot, with `<tmp>` a per-invocation `os.MkdirTemp`, logs the spawn and its end through `logger`, and the child environment gains `PrebuiltLyxEnv=<tmp>/lyx`; the cleanup removes `<tmp>` after `go test` ends.
`buildRoot` is the worktree root inside a hub and `dir` outside one; when `<root>/cmd/lyx` does not exist the build is skipped, so a gate run outside a hub in a repository without `cmd/lyx` builds nothing.
A gate run on a nested module inside a hub still builds the worktree's `lyx` and exports it, which that module's tests may leave unused.
On every path that exports no fresh build, untagged and tagged with the build skipped alike, `gateslot.StripPrebuilt` removes an inherited value from the child environment, so `lyx gate test` always sets or strips the variable itself.
`StripPrebuilt` sits in `gateslot` beside `PrebuiltLyxEnv` because `gatecli`, `cmd/testtiming`, `verifytree` and `websterengine` all call it.
`verifytree.Verify` runs a verify gate's command, the plan-level verify and Publish's `publish_verify` included, as a raw shell command with the inherited environment, so it strips the variable too: the environment it hands `verifyrun.Run` is `StripPrebuilt(os.Environ())`, through `lease.Env` on the slotted path and as it is on the unslotted one, which today passes a nil environment.
Webster's card-verify rerun, `websterengine.runCardVerify`, runs each card's `Verify:` line the same way and strips the variable the same way: `StripPrebuilt(lease.Env(os.Environ()))` on the slotted path, and `StripPrebuilt(os.Environ())` on the unslotted one, which today passes a nil environment.
`verifyrun.Run` has no other caller, so every verify command lyx runs sees no inherited value.
Such a command's own `go test` therefore builds `lyx` once per test binary, as `lyxbin` does with the variable unset.
What the variable lets through and its bound: it lets every package of a tagged gate run skip its own `lyx` build and run the one binary the gate built from the same tree in the same invocation.
A forged value never reaches a gated run, because `lyx gate test` sets or strips it and both verify paths, `verifytree.Verify` and webster's card-verify rerun, strip it, so it can mislead only an ungated raw `go test`.
A build failure is reported on the envelope with the build's output and the exit code 1, before `go test` runs.
`cmd/testtiming`'s `prebuildLyx` does the same for its own runs, with `go` from PATH and the module root: a tagged run builds and exports, and an untagged run, or a tagged one with no `cmd/lyx`, strips an inherited value; `run`, `runRedundancy` and `runResources` call it, the environment it returns reaches each `go test` child, and the cleanup runs after the last child.
Tests: `TestGateTest_Scenario` gains three steps; the stand-in appends one record per invocation (a line of JSON) so a step can read both the build and the test invocation, and `standInRecord` gains a `Prebuilt` field.
Step one, tagged: the stand-in sees a `build -C <prime> -o <path> ./cmd/lyx` invocation before the `test` invocation whose environment carries the variable set to `<path>`, and `<path>`'s directory is gone once the gate exits.
Step two, untagged with the variable set in the gate's own environment: the stand-in sees no build and no variable.
Step three, tagged in a root with no `cmd/lyx` and the variable set in the gate's own environment: the stand-in sees no build and no variable.
`TestVerify_SlotGate`, already not parallel for `GOFLAGS`, also sets the variable with `t.Setenv`, and its command prints it beside `GOFLAGS` and the slot variable; the assertion wants it empty.
`TestRerunCardVerifies_AllPass`, already not parallel, sets the variable with `t.Setenv`, names it as its global state, and gains a card whose command is `test -z "$LYX_PREBUILT_LYX"`, which passes only when the rerun stripped it.
The gatecli doc's `Verb` and `Inheritance` sections describe the build, the variable and the strip, `verifytree`'s doc and `websterengine`'s card-verify paragraph state that a verify command never sees the variable, and `running-tests.md` names the variable and that `lyx gate test --tags` is where one build serves every package.

**ImpactSummary:** `runTest` gains one spawn per tagged run; three harness entry points gain an environment and a cleanup; `Verify`'s command and webster's card-verify rerun each lose one inherited variable, and every verify command keeps its command line; the child's `go test` arguments are unchanged.
