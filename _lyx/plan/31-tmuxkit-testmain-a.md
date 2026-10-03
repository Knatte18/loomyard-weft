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
- `internal/gitkit/testmain_test.go`

**Uses:**
- `internal/testkit/tmuxkit/tmuxkit.go`

**Intent:** Run the first half of the packages that have an `integration`- or `smoke`-tagged test file through `tmuxkit.Main` (D17, D18), under every tag set that compiles their tests.
Each existing `TestMain` keeps `gitkit.HermeticGitEnv()` (or whatever setup it has) first and exits with `tmuxkit.Main(m)` in place of `m.Run()`.
A package whose only `TestMain` is integration-tagged gains a `//go:build !integration` `testmain_test.go` sibling calling `tmuxkit.Main`, keeping the hermetic call where its untagged tests spawn git;
a package with no `TestMain` gains an untagged one.
`gitkit` is such a package: its only `TestMain` sits in the integration-tagged `gitkit_test.go` beside untagged test files, so it gains a `!integration` `testmain_test.go` calling `HermeticGitEnv()` then `tmuxkit.Main`.
`gitrepo`'s only `TestMain` is the integration-tagged `testmain_test.go` itself, beside untagged test files, so the sibling name is taken;
that file drops its `//go:build integration` line instead, so its one `TestMain` compiles under every tag set (`HermeticGitEnv()` only sets environment and is exempt from Tier Purity), and its doc comment, which justifies the tag, is rewritten.
Existing file doc comments are updated to say the binary also runs under tmux isolation.
At HEAD no test in these packages reads `TMUX` or `TMUX_PANE`, so no test relies on running inside the caller's tmux (D17's bound on unsetting `TMUX`);
a test that fails for that reason is reported as a finding, not fixed outside this card's targets.
This card changes no `smoke`-tagged file.

**ImpactSummary:** Test-binary setup only, in the first half of the tagged-test packages; no test body changes.
