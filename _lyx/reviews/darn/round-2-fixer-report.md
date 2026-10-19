# Fixer-report, round 2

## Changes

- F1, fixed as proposed: added three rows to `TestCommandRunsMutatingGit` in `internal/burlerengine/cluster_test.go`: `git --git-dir x commit -m x`, `git --namespace ns commit -m x` and `git --config-env name=VAR commit -m x`, each want true.
  `lyx gate test ./internal/burlerengine` is green.
  Committed as `5c50abf7b`.
- F2, fixed as proposed: reworded `_lyx/landing/summary.md`.
  The help escape now applies only to `-h` or `--help` among git's global options or as the first word after the subcommand.
  `--attr-source` joins the skipped value options.
  The splitter is described as a quote-aware tokenizer that honours backslash escapes, comments and backslash-newline continuations.
  The Design decisions line names the same position bound.
  The file is under `_lyx`, so it is not committed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings; one non-finding note that the joined sentence in the Design decisions line reads slightly awkwardly, left as is.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
