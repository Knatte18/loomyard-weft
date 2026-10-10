# Card 1 — fixture-git-marker

**Create:**
- `internal/gitkit#FixtureGitEnv`
- `internal/gitkit#fixtureGitEnviron`
- `internal/hubforge#markFixtureBuild`
- `internal/hubforge#fixtureBuildMu`
- `internal/hubforge#fixtureBuilds`

**Edit:**
- `internal/gitkit#MustRun`
- `internal/gitkit#Git`
- `internal/gitkit#gitExit`
- `internal/gitkit#GitStatusPorcelain`
- `internal/gitkit#mustGit`
- `internal/hubforge#mustGit`
- `internal/hubforge#NewHub`
- `internal/hubforge#AddPairWith`
- `internal/hubforge#buildBareTemplate`
- `internal/gitkit/gitkit_test.go`
- `internal/hubforge/hub_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Prosa:**
- `internal/gitkit/doc.go`
- `internal/hubforge/doc.go`

**Intent:** Fixture git carries a marker so the census of card 3 can tell fixture git from code under test (D2).
`FixtureGitEnv` is the one declaration of the marker variable's name, in `gitkit`; its value is `1` on a marked child.
`fixtureGitEnviron` returns the process environment with the marker appended, and every spawn helper in `gitkit.go` (`MustRun`, `gitExit` behind `Git`, `GitStatusPorcelain`, `mustGit`) and hubforge's `mustGit` set it on the child's `Env`.
A hub build spawns its git through production `fabriccli.CloneAndWire` and `gitexec`, which no helper reaches, so hubforge sets the marker in the process environment for the build's duration: `markFixtureBuild` increments `fixtureBuilds` under `fixtureBuildMu`, sets the variable when the count goes from zero to one, and returns a release function that decrements and unsets it when the count returns to zero; `NewHub`, `AddPairWith` and `buildBareTemplate` each hold one for their body.
Builds stay concurrent: only the first starter sets and the last finisher unsets.
Tests, both in the existing integration-tagged files: `gitkit_test.go` gains `TestFixtureGitMarker_ReachesHelperChildren`, which runs `Git` with `GIT_TRACE2_EVENT` pointed at a temp directory and `GIT_TRACE2_ENV_VARS` set to the marker's name through `t.Setenv`, and asserts the event file's `def_param` event names the marker, the same mechanism card 3 reads; `TestNewHub` gains an assertion that the marker is unset in the process environment once `NewHub` returns, and `TestNewHub_Concurrent` asserts the same after all builds, so the counter is pinned.
The test uses `t.Setenv` and so does not call `t.Parallel`; its comment names the process environment as the global state.
`gitkit`'s row in the budget rises by one.
Both package docs name the marker and the two ways it is set.

**ImpactSummary:** Five spawn helpers and three hub-build functions gain an environment line; no caller's signature changes and no production `lyx` code is touched.
