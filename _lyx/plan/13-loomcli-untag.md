# Card 13 — loomcli-untag

**Edit:**
- `internal/loomcli#`

**Intent:**
Run the untag procedure (Shared Decisions, `untag-procedure`) on `internal/loomcli`'s `integration` and `tmux` files.

The `integration` files (`wiring_*`, `landingdeps`, `commitrecords`, `circling`, `parentreview`, `reviewrange`) wire a hub through `hubforge`; a test whose assertion is about loomcli's own output envelope, refusal text or the seams it passes to its engines, and not about the git state, moves to an untagged file over the injected seams `newLoomCLI` already takes, or a seam added as a struct field or option with production passing today's value (D6).
The `smoke_*` and `integration_driverbootstrap_test.go` files drive a real reed session and the built `lyx`; they stay `tmux` unless a test's behavior needs neither.

Then merge scenario families (Shared Decisions, `scenario-merge`) among the tests that stay tagged — `wiring_commitstatus_integration_test.go`'s tests (`integration`) and the `smoke_test.go` family (`tmux`) are the likely ones, each merged only with tests of its own tier.
The commit message names each merged family and how many fixture builds it removed.

The "Stub driver helpers stay untouched" decision binds this card: `writeStubDriverScript` in `integration_driverbootstrap_test.go` and `integrationWriteStubDriverScript` in `smoke_driverstrand_test.go` keep their names, signatures and bodies, and nothing changes that would require editing them.
Tests in those two files may still move, fold or merge around them.

`TestMain` keeps `tmuxkit.Main` and `gitkit.HermeticGitEnv()` while tagged files remain.

**ImpactSummary:** Test files of one package, plus a seam field in its production code only where a moved test needs one; production passes today's value, so `lyx loom` behaves as before.

**Verify:**
go test -tags tmux -count=1 ./internal/loomcli/
go vet -tags llm ./internal/loomcli/
