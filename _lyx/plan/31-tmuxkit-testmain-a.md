# Card 31 — tmuxkit-testmain-a

**Edit:**
- `cmd/lyx/testmain_test.go`
- `internal/battencli/testmain_test.go`
- `internal/battencli/testmain_integration_test.go`
- `internal/boardcli/testmain_test.go`
- `internal/boardengine/testmain_test.go`
- `internal/boardengine/boardtest/testmain_test.go`
- `internal/burlercli/testmain_test.go`
- `internal/burlerengine/testmain_test.go`
- `internal/configcli/testmain_test.go`
- `internal/fabriccli/testmain_test.go`
- `internal/fabricengine/testmain_test.go`
- `internal/gitexec/testmain_test.go`
- `internal/gitkit/gitkit_test.go`
- `internal/gitrepo/testmain_test.go`
- `internal/hubforge/testmain_test.go`
- `internal/hubgeom/testmain_integration_test.go`
- `internal/idecli/testmain_test.go`
- `internal/ideengine/testmain_test.go`

**Create:**
- `internal/cliwire/testmain_test.go`
- `internal/hubgeom/testmain_test.go`

**Uses:**
- `internal/testkit/tmuxkit/tmuxkit.go`

**Intent:** Run the first half of the packages that have an `integration`- or `smoke`-tagged test file through `tmuxkit.Main` (D17, D18), under every tag set that compiles their tests.
Each existing `TestMain` keeps `gitkit.HermeticGitEnv()` (or whatever setup it has) first and exits with `tmuxkit.Main(m)` in place of `m.Run()`.
A package whose only `TestMain` is integration-tagged gains a `//go:build !integration` `testmain_test.go` sibling calling `tmuxkit.Main`, keeping the hermetic call where its untagged tests spawn git;
a package with no `TestMain` gains an untagged one.
Existing file doc comments are updated to say the binary also runs under tmux isolation.
Any test these packages hold that relies on running inside the caller's tmux is fixed to set the variable it needs (D17's bound on unsetting `TMUX`).
This card changes no `smoke`-tagged file.

**ImpactSummary:** Test-binary setup only, in the first half of the tagged-test packages; no test body changes.
