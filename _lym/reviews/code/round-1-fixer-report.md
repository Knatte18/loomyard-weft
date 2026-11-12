## Fixer-report, round 1

All seven findings are fixed.
`lym gate test ./internal/gatecli --tier 1` passed (0 failed) after the main edits;
the later edits touched only a help string, a comment and a research report, and `go vet` of both packages is clean.

## Changes

- **F1** (as proposed): `TestGateRedundant` gains two refusal rows.
  `badindex` holds a map whose test names block 5 of a one-block table, wanting "is unreadable".
  `badkeep` holds a sound map and a `//testtiming:keep` line with no reason, wanting "the keep scan of".
  Both also want "way forward".
- **F2** (as proposed): the answer carries `map_build_failure` with the commit and error when `m.Failure` is set, and omits it otherwise.
  `doc.go` and the verb's `Long` help name it.
  The happy-path test map carries a `Failure` and the test pins the key.
- **F3** (as proposed): `docs/research/README.md` indexes the coverage-map pilot report.
- **F4** (as proposed, then trimmed): the `JudgeMap` doc comment no longer refers to `Judge`.
  The "counts only judged entries as covers" clause the review suggested repeated the keep and outcome sentences below it, so it was dropped.
- **F5** (as proposed): `TestCovSub` is tier 1 in the written map and the wanted candidate.
- **F6** (as proposed): the way forward reads "declare tier0 or tier1 with its dir at the Go module's root with lym config gate from the prime, then re-run the same command".
- **F7** (fixed differently): the review proposed a clause explaining 35 tests against 34 entries.
  Nothing in the report or the code I read explains the gap, so I took the review's other option and dropped the `tests` figure from the row.

A gate finding then flagged a 120-character comment line in `internal/gatecli/doc.go` (the `map_build_failure` sentence from F2).
A first split at the comma was flagged as a wrapped break, so the sentence is now two short sentences, each on one line, in `gatecli: fix gate — rewrite the doc.go map_build_failure sentence`.

Commits, newest first (the gate commit above them all): `docs: fix F7`, `testcover: fix F4`, `gatecli: fix F2`, `docs: fix F3`, `testcover: fix F4`, `gatecli: fix F1, F2, F5, F6`.
F1, F2, F5 and F6 share one commit: they edit the same two files and the same test table.

## Self-check

- fix-consistency, LOW: the `Long` help did not name `map_build_failure`. Fixed as proposed.
- fix-proposal, LOW: the F7 clause restated the gap without a cause. Fixed by dropping the figure.
- fix-proposal, LOW: the F4 clause repeated the lines below it. Fixed by dropping the clause.
- fix-proposal, LOW: `testmap.go:375` words the module way forward differently from the F6 text. Disputed, premise false (see Disputed).
- fix-affected-set, LOW: the same `testmap.go:375` twin. Disputed, same ground.
- fix-affected-set: seven `speed/` reports were already unindexed before this round. Not changed, since the finding asked for the new report only.
- fix-code-correctness: no findings.

## Disputed

- **`testmap.go:375` as the twin of F6** (lens findings, premise false):
  `testmap.go:372` takes the minimum key of `r.tiers`, so the tier set is non-empty there, and the message fires only when that tier's dir holds no go.mod.
  "set the dir of the lowest of tier0 and tier1" is the right step on that path.
  The no-tier cause F6 names reaches only `redundancyModule`.

## Deferred

None.

## Rulings

None.
