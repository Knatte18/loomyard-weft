---
format: 5
approved: true
language: go
---

# Plan: test-suite-prune-rest

Shrink the test suite of every Go package outside `test-suite-prune-core`'s set without losing a pinned contract.
The plan records the before measurement on the untouched tree, then prunes the in-scope packages one group per card: fix tests that fail alone, move `integration` tests that spawn nothing to the untagged tier, merge fixture families into scenarios, fold near-identical tests into tables, and delete or keep each candidate of `docs/benchmarks/test-redundancy.md`.
It ends with a per-package rerun spliced into the report and the after measurement.

## Card Index

1 — measure-before — record the before measurement in a committed benchmark report
2 — cmd-lyx-prune — prune cmd/lyx's tests, keeping every guard self-check
3 — tooling-contracts-prune — prune contracts and tools tests
4 — parsers-prune — prune the sole-parser packages' and mergeresolve's tests
5 — planglyph-prune — prune planglyph's tests
6 — logger-prune — prune logger, loggerconfig and output tests
7 — yaml-config-prune — prune the YAML and config packages' tests
8 — landingshed-prune — prune landingshed's tests
9 — batten-prune — prune the batten packages' tests
10 — board-prune — prune the board packages' tests
11 — loom-prune — prune loomengine, loomrecipe and loomshed tests
12 — shed-prune — prune the shed packages' tests outside shedadapters
13 — review-gates-prune — prune the burler, treadle, review, preflight, verify and status-commit packages' tests
14 — reed-prune — prune reedcli and reedengine/render tests
15 — shuttle-prune — prune shuttlecli and claudeengine tests
16 — webster-orch-prune — prune webstercli, orchcli and orchengine tests
17 — fabric-hub-prune — prune the fabric CLI, hub fixture and pair teardown tests
18 — geometry-prune — prune the hub, standalone and cwd geometry packages' and weftname's tests
19 — git-fs-prune — prune the git access, filesystem, lock and process packages' tests
20 — stencil-state-prune — prune the stencil, state, cliwire and agentname packages' tests
21 — cli-ide-prune — prune the small CLI and IDE packages' tests
22 — friction-leaf-prune — prune the friction, leaf, registry and seam packages' tests
23 — rerun-measure-after — rerun and splice the in-scope report sections and record the after measurement

## Shared Decisions

### Decision: ownership-boundary

- **Decision:** No card edits a file under `internal/fabricengine`, `internal/websterengine`, `internal/reedengine`, `internal/shuttleengine`, `internal/loomcli`, `internal/shedadapters` or `internal/testkit/`, nor an `llm`-tagged file.
  A sub-package with its own directory below a core package (`internal/reedengine/render`, `internal/shuttleengine/claudeengine`) is this task's.
  A test that a core or testkit file names is never deleted, renamed, folded away or merged away, and a reference that would need an edit there is left alone and listed in the commit message, for the PR body (Decision 1).
  No card edits a file under `cmd/testtiming`, tests included.
  Production code does not change, except deleting a non-test helper that pruning leaves with no caller.
  No test that exercises skill loading is edited, folded, merged or deleted: the tests naming `Spec.Skills`, `SkillLoadSequence`, `SkillUnknown`, `DefaultSkillLoadTimeout`, `loadSkill` or `loadSkillsThenPrompt` in `internal/shuttleengine/claudeengine`, and the `SkillLoader` fakes they use.
  Each keeps its own name as a top-level test, and a redundancy candidate among them is kept with `//testtiming:keep` naming `skills-in-one-turn`.
- **Rationale:** Decision 1 and the decision record's Scope, and its parent additions on `cmd/testtiming` and on skill-loading tests; the two prune runs work in parallel on disjoint files, and the parallel task `skills-in-one-turn` rewrites skill loading and its tests.
- **Applies to:** cards 2–23

### Decision: prune-procedure

- **Decision:** A prune card runs, for each package it targets, in this order:
  1. `go run ./cmd/testtiming -redundancy -tags integration,tmux -pkg ./<package> -out .scratch/<name>.md` refreshes that package's candidate list into a scratch file; `-out` always points under `.scratch/`, never at the committed report, which only `rerun-measure-after` rewrites (Decision 9).
  2. Each `failed` entry is rerun alone and judged under Decision 8, before any fold or merge touches it; a test that fails alone is fixed in the test, and a tool artifact is named in the commit message.
  3. Tier moves (Decision 5): a test in an `integration` file that needs neither git, tmux nor a subprocess moves to an untagged file, using an existing `internal/testkit` fake or a package-local one; a move that needs a new shared fake is not made, and the test gets the `// tiermove: needs shared fake <name>` comment.
  4. Scenario merges (Decision 13 as amended by the parent's three "Added after Discussion" scenario decisions): tests of one tier that each build the same `hubforge` hub or pair, tmux server or seeded worktree become one scenario whose named `t.Run` steps run serially in one order, `if !t.Run(...) { return }` stopping it at a failing step; a step that relies on an earlier step's state says so in a comment, and the scenario's top-level test calls `t.Parallel` with a comment naming the shared fixture as the reason no step does.
     A family is merged when it removes fixture builds, and the commit message names each merged family and the builds it removed.
     Cheap unit tests are not merged into scenarios; they stay table-driven.
     After a merge, a `failed` entry's alone-rerun is `go test -count=1 -tags integration,tmux -run '^<Scenario>$' <package>` on the scenario as a whole, never on one step.
  5. Folds (Decision 4): every test file of the package is read, candidates and their covering tests first; tests that differ only in inputs and expected values become one table-driven test named after the behavior, with `t.Parallel` at the top level and per subtest unless a comment names the global state.
  6. Delete or keep (Decisions 2, 3 and 10): a candidate is deleted only when it is `Removable: yes` and a read of its assertions against its covering tests finds nothing unique, or its unique assertions move into a covering test first; otherwise it is folded, merged or kept with `//testtiming:keep <reason>` directly above its `func Test…` line.
     The reason is non-empty and is exactly one of these classes, the four of Decision 10 and the skill-loading reason of `ownership-boundary`:
     what the test pins that its covering tests do not (a guard that fires, an error message, an output shape, or a kept partner of a removed test);
     the core or testkit file that names the test;
     the assertion it received and the test it came from;
     for a `Removable: no` candidate that no fold or merge applies to, the covering test the report ties it to;
     for a skill-loading test that `ownership-boundary` lists, the name `skills-in-one-turn`, and for no other test.
     A candidate with none of these reasons is deleted or folded, never annotated: no other text satisfies the loop.
     The commit message lists every deleted candidate with its receiving test or "no unique assertion", every kept candidate grouped by reason class, and each `// tiermove:` test with its fake.
  7. Leftovers and references (Decisions 6 and 7): a test helper no file calls under any tag, `llm` included, is deleted, and so is a test file left empty; a `TestMain` stays.
     A doc, `pattern/` background, stencil, spec or comment outside the core directories that names a deleted or renamed test is updated in the same commit.
  8. Repeat from step 1 until the scratch list shows only candidates that carry a keep directive whose reason is one of the classes of step 6.

  Every protected test (Decisions 1 and 2) keeps its own name as a top-level test: it may host a fold table, but is never a scenario step.
  Every fold, merge and move keeps every assertion it replaces, and the package keeps `gitkit.HermeticGitEnv()` and `tmuxkit.Main` where `PATTERN-test-isolation` requires them.
  Comments written or changed follow `PATTERN-comment-line-breaks`.
- **Rationale:** Decisions 2–10 and 13, `PATTERN-test-economy` and `PATTERN-test-speed`; each package is judged against its own tests only, so a package card is self-contained.
- **Applies to:** cards 2–22, and card 23 for a gap its final rerun finds

## verify:

go build ./...
go vet ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test -count=1 ./...
go test -count=1 -tags integration ./...
go test -count=1 -tags tmux ./...
