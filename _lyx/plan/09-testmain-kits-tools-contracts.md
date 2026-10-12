# Card 9 — testmain-kits-tools-contracts

**Create:**
- `internal/testkit/testmain_test.go`
- `internal/testkit/envelope/testmain_test.go`
- `internal/testkit/envkit/testmain_test.go`
- `internal/testkit/llmkit/testmain_test.go`
- `internal/testkit/locationkit/testmain_test.go`
- `internal/testkit/logcapture/testmain_test.go`
- `internal/testkit/plankit/testmain_test.go`
- `internal/testkit/scankit/testmain_test.go`
- `internal/testkit/shedfake/testmain_test.go`
- `internal/testkit/shuttlefake/testmain_test.go`
- `internal/testkit/stencilkit/testmain_test.go`
- `tools/codestats/testmain_test.go`
- `tools/deploy/testmain_test.go`
- `tools/godocreflow/testmain_test.go`
- `tools/internal/devbin/testmain_test.go`
- `tools/mdreflow/testmain_test.go`
- `tools/sandbox/testmain_test.go`
- `tools/wordswap/testmain_test.go`
- `contracts/specs/testmain_test.go`
- `contracts/stencils/testmain_test.go`

**Edit:**
- `internal/gitrepo/internal/gitoracle#TestMain`

**Uses:**
- `internal/testkit/tmuxkit#Main`

**Intent:**
The same change as cards 7 and 8 for the remaining test packages with no `TestMain`, under `internal/testkit/`, `tools/` and `contracts/`: an untagged `testmain_test.go` whose `TestMain` is `os.Exit(tmuxkit.Main(m))`, with no build constraint, the package clause the package's test files use, and a one-line comment saying why it exists (D4).
A kit's test file importing `tmuxkit` is allowed: the Testkit Invariant bans that import from a kit's non-test files only, and none of these kits is imported by `tmuxkit`, so no cycle arises.

`gitoracle`'s `TestMain`, the one existing `TestMain` that does not call the entry point, keeps `gitkit.HermeticGitEnv()` first and runs the tests as `os.Exit(tmuxkit.Main(m))` instead of `os.Exit(m.Run())`, so its run directory, created by `HermeticGitEnv`, is removed after the run; its file comment says so.
After this card every test package's `TestMain` calls `tmuxkit.Main`, which card 10's scan then requires.

**ImpactSummary:** gitoracle's TestMain now runs its tests through tmuxkit.Main; no other code changes.
