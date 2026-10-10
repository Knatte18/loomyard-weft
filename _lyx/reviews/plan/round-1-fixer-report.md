# Plan-Burler round 1 — fixer report

## Fixed

- **F1** (`02-classifier-shell-words.md`): the Intent now states that every `git` word `admits` does not judge rejects an unquoted glob: `readOnlyGitSegment` in the `-C` directory and the `cat-file` and `config` words, and `readOnlyGitBranch` in a `--list` pattern, where a quoted glob stays text.
  A bound sentence says an unquoted glob reaches git only after `--` in `log`, `show`, `diff` and `status`.
  Rejected rows added: `git branch --list *`, `git -C * status`, `git cat-file -p *`, `git config --get *`.
- **F2** (`08-thin-table-rows.md`): confirmed against `internal/shuttleengine/wait.go`, where `countsAtOnce` returns true for any ungated run and `endedShells` records every counted shell with no awaited filter.
  The awaited row now runs gated and fresh under a matching `AwaitedShellPrefixes`, with every output present: the waiting turn end does not count, a `multiStepClock` step appends a `STOP:` line with nothing outstanding, and the run finishes `OutcomeDone` through the passing gate with `EndedShells` empty.
  The card says why the ungated finish cannot carry the criterion, and names the columns the row adds (expected `EndedShells`, gate, started-fresh mark, awaited prefixes, clock steps).
- **F3** (`02-classifier-shell-words.md`): the duplicate rows `git log a?`, `git log [a]`, `git log {a,b}` and the second `git log ~/x` are gone.
  The same reasoning drops `git log *.go`, which the existing "git log with a glob star" row (`git log *`) already pins: an unquoted glob before `--` in `log`.
  One sentence names the existing rows that cover these cases, and the flipped home-path row names its command.
- **F5** (`06-merriam-base-one-way-forward.md`): the ImpactSummary now says three of `MerriamBase`'s four callers gain their own way forward and `autoRebaseline`'s existing transient wrapper supplies the fourth's.

`lyx loom validate-plan` reports `ok: true` over the edited plan.

## Disputed

- **F4** (Card 4's budget premise): the premise that `cmd/lyx` holds 71 top-level tests is false.
  `grep -hE '^func Test' cmd/lyx/*_test.go` lists 73 functions.
  One is `TestMain`, which `isTopLevelTest` (`cmd/lyx/testbudget_test.go:31`) excludes, so the package counts 72 tests.
  That is exactly its row in `cmd/lyx/testdata/test-budget.yaml` (`cmd/lyx: 72`).
  The reviewer's 71 most likely came from filtering out every line containing `TestMain`, which also drops `TestHermeticGitEnv_GitSpawningPackagesHaveTestMain` (`cmd/lyx/hermeticenv_test.go`).
  `TestOwnJSONFlags` brings the count to 73, over the row, so Card 4 has to raise the row in the same card, as CLAUDE.md's test-budget rule requires.
  Card 4 is unchanged.

## Deferred

None.
