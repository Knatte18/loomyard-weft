# Card 11 — tmux-isolation-guard-rename

**Rename:**
- `cmd/lyx#TestTmuxIsolation_TaggedPackagesRunThroughTmuxkitMain` -> `cmd/lyx#TestTmuxIsolation_EveryTestPackageRunsThroughTmuxkitMain`

**Edit:**
- `cmd/lyx#TestFileNamePlatform`

**Intent:**
Rename the tmux-isolation guard so its name states what card 10 made it check: every test package, not only tagged ones, runs through `tmuxkit.Main`.
The rename rewrites the guard's own doc comment opening, and `TestFileNamePlatform`'s `//testtiming:keep` reason, which names the guard, names it by its new name.
A grep for the old name finds no other live reference; `docs/benchmarks/test-redundancy.md` is a measurement report of one past run and keeps the name it recorded.
Pure rename: the guard's existing run covers it.

**ImpactSummary:** One test function renamed and one keep-reason comment retargeted; no behavior changes.
