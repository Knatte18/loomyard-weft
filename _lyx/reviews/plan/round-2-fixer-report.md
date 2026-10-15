# Fixer report — plan, round 2

Every finding's premise held against the cards, and every finding was fixed as proposed.
`lyx loom validate-plan` passes after the edits, with informational findings only.

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  `_lyx/plan/01-cli-and-output.md`: the variable-block bullet gains the sentence deriving the run-record root and the logs root as the first path segment of the plan and logs directories, by bash parameter expansion, and handing the plan directory and both roots to the jq filters with `--arg`.
  `_lyx/plan/02-fork-records.md`: the `extract.jq` paragraph says the card count comes from card lines under the plan directory passed with `--arg`, not a spelled `_lyx/plan/` pattern.
  `_lyx/plan/05-pack-check.md`: the `touched-files.jq` bullet skips paths under the run-record root or the logs root passed with `--arg`, and absolute paths outside the worktree.
- **F2 (MEDIUM), fixed as proposed.**
  `_lyx/plan/04-trace-cross-check.md`: `plan:tools/forkaudit#TestForkauditReport` added to `**Edit:**` after `runForkaudit`, and the Tests section gains the bullet that the run-twice row passes one `--records` path, computed once per row, on both runs.
  The `runForkaudit` default for callers passing no `--records` is unchanged.
- **F3 (LOW), fixed as proposed.**
  `_lyx/plan/02-fork-records.md`: the Samples bullet names `file`, `raf`, `inline`, `none` and `unknown` as the arms named when empty, and says a `trace:<value>` arm exists only with forks.
- **F4 (LOW), fixed as proposed.**
  `_lyx/plan/01-cli-and-output.md`: the permission-denial line gains the `t.Cleanup` that restores mode 0o755 on the directory a row made read-only.
- **F5 (NIT), fixed as proposed.**
  `_lyx/plan/01-cli-and-output.md`: the header-comment rule now covers cards that add a flag or an optional section.

## Disputed

None.

## Deferred

None.
