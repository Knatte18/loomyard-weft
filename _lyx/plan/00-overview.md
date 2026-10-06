---
format: 5
approved: false
language: go
---

# Plan: test-suite-prune-core

Shrink the test suites of `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli` and `internal/shedadapters` without losing a pinned behavior.
First make `cmd/testtiming`'s redundancy mode able to judge these packages (a flag-form fix for single-test runs, D1's narrower exclusion, D2's keep directive, D3's section splice), record the before measurement, regenerate the six report sections, then prune each package: move tests that need no real substrate to the untagged tier, merge fixture families into scenarios, fold near-identical tests into tables, and delete or keep each remaining candidate.
The plan ends with the after measurement and a final rerun of the six sections.

## Card Index

1 — run-alone-flag-form — pass every flag of a single-test coverage run in -flag=value form
2 — exclusion-out-of-process — exclude a test only when module code may run out of process (D1)
3 — keep-directive — read //testtiming:keep and list kept tests with their reasons (D2)
4 — report-splice — comma-separated -pkg and a narrowed run that rewrites only its own sections (D3)
5 — measure-before — record the before measurement in a committed benchmark report (D8)
6 — redundancy-rerun — regenerate the six packages' report sections with the new tool
7 — shedadapters-prune — fold, delete or keep shedadapters' tests
8 — shuttleengine-prune — fold, delete or keep shuttleengine's tests
9 — websterengine-untag — move websterengine tests off real git and merge fixture families
10 — websterengine-prune — fold, delete or keep websterengine's tests
11 — reedengine-untag — move reedengine tests off a real tmux server and merge fixture families
12 — reedengine-prune — fold, delete or keep reedengine's tests
13 — loomcli-untag — move loomcli tests off real git and tmux and merge fixture families
14 — loomcli-prune — fold, delete or keep loomcli's tests and fix tests that fail alone
15 — fabricengine-untag — move fabricengine tests off real git and merge fixture families
16 — fabricengine-prune — fold, delete or keep fabricengine's tests
17 — measure-after — record the after measurement and rerun the six sections (D8)

## Shared Decisions

### Decision: untag-procedure

- **Decision:** An untag card reads every `integration` or `tmux` test of its package.
  A test whose behavior does not depend on real git, tmux or a process moves to an untagged file and runs against a fake or in-memory fixture (D6).
  A fake used by one package stays in its `_test.go` files; a fake also needed by another of the six goes under `internal/testkit/<kit>/` (`PATTERN-testkit`), which is the one change outside the card's own package an untag card may make.
  A missing seam is added as a struct field or option, never a package-level variable, with production passing today's value.
  A moved test passes `cmd/lyx`'s `tierpurity_test.go` and `PATTERN-test-isolation`, or it stays tagged.
  When the package's last tagged test file goes, its `TestMain` may drop `tmuxkit.Main`; a package that still spawns git keeps `gitkit.HermeticGitEnv()`.
  Then the card merges scenario families per `scenario-merge`.
- **Rationale:** D6 and the operator's scenario decision; doing the tier move before the prune lets the prune card judge the moved tests, since untagged tests are always judged (D1).
- **Applies to:** cards 9, 11, 13, 15

### Decision: scenario-merge

- **Decision:** Tests of one tier that each build the same expensive fixture (a `hubforge` hub or pair, a tmux server, a seeded worktree) and differ only in the steps and checks after it become one scenario test that builds the fixture once.
  Each former test becomes a named `t.Run` step; the steps run serially, in one order, and may build on earlier steps' state, and a step that relies on an earlier step's state says so in a comment.
  A family is merged only when its tests can run in that one order on one fixture; tests that each need a pristine fixture stay separate.
  A scenario never mixes tiers, and a merge never moves a test to a higher tier.
  A merge is justified by the fixture builds it removes, not by per-family timing; the commit message names each merged family and the builds it removed.
  Every assertion of the merged tests is kept.
  Cheap unit tests are not merged into scenarios.
  The scenario test itself calls `t.Parallel` unless it touches process-global state, which its comment names.
- **Rationale:** The operator's scenario decision as amended by the parent's "A scenario stays within one tier", "Scenario steps share state by design" and "Scenario merges are judged by fixture builds, not per-family timing".
- **Applies to:** cards 9, 11, 13, 15

### Decision: prune-procedure

- **Decision:** A prune card runs, for its package:
  1. `go run ./cmd/testtiming -redundancy -tags integration,tmux -pkg ./internal/<pkg>` to refresh its section.
  2. Fold (D5): tests that share setup and differ only in inputs and expected outputs become one table-driven test, judged by reading every test of the package, including the ones the tool excludes.
     Each former test survives as a named `t.Run` subtest, and the folded test keeps `t.Parallel`.
  3. Delete or keep (D4): a candidate is deleted only when it is in the current removable set and a read of its assertions finds nothing its covering tests do not also assert, and it does not make git run module code through a hook, alias or helper it installs.
     A candidate whose assertions add something is folded into its covering test as an assertion or row, or kept with `//testtiming:keep <reason>` naming what it pins that its covering tests do not.
  4. A test listed as "failed" is fixed so it passes alone (D7).
  5. Rerun step 1 and repeat 2–4 until the section lists no candidate outside "Kept" and no test as "failed", then commit the test changes with the regenerated section.
  The six packages' test files are this task's; no other package's test file is edited (`test-suite-prune-rest` owns them), except a shared kit under `internal/testkit/` per `untag-procedure`.
- **Rationale:** D4, D5 and D7, and `PATTERN-test-economy`'s rule that only the removable set may be deleted together.
- **Applies to:** cards 7, 8, 10, 12, 14, 16

## verify:

go build ./...
go vet ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test -count=1 ./...
go test -tags integration -count=1 ./cmd/lyx/ ./cmd/testtiming/ ./internal/fabricengine/ ./internal/websterengine/ ./internal/reedengine/ ./internal/shuttleengine/ ./internal/loomcli/ ./internal/shedadapters/ ./internal/testkit/...
go test -tags tmux -count=1 ./cmd/lyx/ ./cmd/testtiming/ ./internal/fabricengine/ ./internal/websterengine/ ./internal/reedengine/ ./internal/shuttleengine/ ./internal/loomcli/ ./internal/shedadapters/ ./internal/testkit/...
