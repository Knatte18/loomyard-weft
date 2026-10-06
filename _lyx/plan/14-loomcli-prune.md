# Card 14 — loomcli-prune

**Edit:**
- `internal/loomcli#`
- `docs/benchmarks/test-redundancy.md`

**Intent:**
Run the prune procedure (Shared Decisions, `prune-procedure`) on `internal/loomcli`, starting with a rerun of `-pkg ./internal/loomcli`, since card 13 moved and merged tests.

D7: card 1 fixed the tool fault behind the nine tests the committed report lists as "failed"; when card 6's or this card's rerun still lists one, fix the test so it passes alone (for example a cobra command that reads the process's arguments gets explicit `SetArgs`).
`TestBuildLoomShed_OutputShape` is listed as skipped; read why, and leave it skipped only when its skip condition is a missing substrate rather than a test-order dependence.

Fold first: the `TestDriverPrompt_*` family, the `TestDriverResumeLine_*` pair, the `TestResolveRunID_*` refusals and the `TestDriverStrandDisplayName_*` pair in the committed report are likely tables, judged by reading.
Then delete or keep the remaining candidates, rerunning until no candidate outside "Kept" remains and no test is "failed".

The "Stub driver helpers stay untouched" decision binds this card as it binds card 13, and so does `skill-loading-tests-untouched`: a candidate that exercises the stub driver scripts' skill loading is kept with `//testtiming:keep` naming `skills-in-one-turn`.

**ImpactSummary:** Test files of one package and its report section; production code is not expected to change.

**Verify:**
go test -tags tmux -count=1 ./internal/loomcli/
