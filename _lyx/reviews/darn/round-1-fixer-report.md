# Round 1 fixer-report

## Changes

All changes are rows in `TestCommandRunsMutatingGit` (`internal/burlerengine/cluster_test.go`); no production code changed.

- F1 (`6a4944a25` lineage, commit `c719ff556`): added a `git tag -l --format <value>` row.
  Fixed differently from the proposal: the proposed value `-x` is two characters and never reaches the short-cluster case, so the row passed with `--format` removed from `gitTagValuedListOptions`.
  The final row uses `-version:refname`, which fails without the skip (see Self-check).
- F2 (`6069e8cce`): added `git tag -l -- v1`, expected not mutating, pinning the `--` branch with list mode set.
- F3 (`6a4944a25`): added rows pairing `-l` with `--delete`, `--annotate`, `--sign`, `--force`, `-u`, `--local-user`, `-F`, `--file`, `-e` and `--edit`, each expected mutating.
- Self-check follow-up: corrected the F1 value, and added `-a`, `-s`, `-f` and `--message=` beside `-l`, each expected mutating.

`lyx gate test ./internal/burlerengine` is green after each commit.

## Gate repair

The tier-1 gate failed `TestGofmt_ModuleFilesAreFormatted`: two added rows lacked the space after the name's comma.
`lyx loom fmt internal/burlerengine/cluster_test.go` fixed both, committed on its own.

## Self-check

- fix-code-correctness, MEDIUM: the F1 row with `-x` passes with the defect in place. Fixed with own change (value `-version:refname`).
- fix-proposal, MEDIUM: same finding as above, merged. Fixed with own change.
- fix-affected-set, LOW: `-a`, `-s`, `-f` and `--message` had no row beside a list option. Fixed as proposed.
- fix-consistency: no findings.

## Disputed

None.

## Deferred

None.
