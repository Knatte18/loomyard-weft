# Fixer report — plan, round 3

## Changes

- **F1** (MEDIUM, card 14) — fixed differently.
  Card 14's Create list gains `plan:internal/gitexec#RunWithEnv`, and its Intent says that each plumbing command (`read-tree`, `update-index`, `write-tree`, `commit-tree`) over the `lyxtemp` index runs through `gitexec.RunWithEnv` with `GIT_INDEX_FILE`, and that the plumbing stays in `fabricengine`.
  It also says that `RunWithEnv` is `Run` with `extraEnv` appended after `gitexec`'s own variables, and that "Only once the result is whole does `integrateWeft` move the branch to it through `resetWeftToIntegrated`".
  Not made: the proposed `gitrepo.IndexEdit` type and `Repo.CommitIndexEdits` method.
  `fabricengine` already runs this plumbing through `gitexec.Run`: `commit-tree` at `internal/fabricengine/snapshotuncommitted.go:207` and `read-tree` at `internal/fabricengine/forkrecords.go:45`.
  `PATTERN-fabric-git` keeps git on a weft in `fabricengine`.
  A one-caller exported `gitrepo` type and method adds an abstraction the premise does not need; the missing seam is only the environment variable.
  The self-check added `**Edit:**` targets `internal/gitexec_test#TestRun`, `#TestRun_StdoutOnError` and `#TestCheckFloor`, test rows for `RunWithEnv`, and an `**ImpactSummary:**` that `validate-plan` requires once a card has Edit targets.
- **F2** (LOW, card 8) — fixed with an adjusted text.
  A sentence after the `retireBoltAnchorMarker` paragraph places it in `internal/fabricengine/anchormarker.go`, which card 6 creates, rather than "the new" file.
  It deletes both files through `root.Remove` on an `os.Root` at the board worktree, so no banned token appears and no allowlist entry is needed.
  The self-check made one further change, below: the deletion runs inside one `BoltWrite` handed to `Bolt.PullThenCommitWritten`.
- **F3** (LOW, card 17) — fixed as proposed.
  The fresh-run row of `TestChangeAnchor` now lists the pass-wait, parent-lock, dobby-end and log-sink-close errors.
  A new row covers a sink-reopen error after a successful `Apply`.
- **F4** (LOW, card 17) — fixed as proposed, then reworded by the self-check (fix-proposal 1 and 2 below).
- **F5** (NIT, card 13) — fixed as proposed: `RemoveMoveRecord`'s callers now name `Close`.

`lyx loom validate-plan` reports `ok` after all changes.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal 1 (LOW, card 17 owner refusal prescribes `--keep-local` every time) — fixed as proposed.
  The refusal now names plain `lyx adopt <slug>`, and `--keep-local` only when adopt stops over an `anchor_stopped` weft.
- fix-proposal 2 (LOW, the prime's `adopt --prime` step is a redundant hop) — fixed with own change.
  The prime clause is kept, since `adopt --prime` is the way forward when the prime weft is not stopped.
  It now says the refusal over a stopped prime weft names the `--set` below, instead of restating that `--set`.
- fix-proposal 3 (LOW, card 8 pinned `Bolt.CommitWritten`, which never pulls first) — fixed with own change.
  The deletion now runs as one `BoltWrite` through `Bolt.PullThenCommitWritten`, the documented sibling for callers that must not write over a stale copy (`internal/fabricengine/bolt.go`).
  A skipped write keeps the marker and is reported `kept` with the skip's detail.
- fix-affected-set 1 (LOW, `RunWithEnv` missing from `gitexec`'s entry-point test tables) — fixed with own change.
  Card 14 edits `TestRun_StdoutOnError`'s `forms` map and `TestCheckFloor`'s every-entry-point row.
  The `extraEnv` row went into `TestRun` beside its environment subtests, rather than into `TestRun_StdoutOnError`.
- fix-affected-set, outside its lens (card 15:64 removes `_launchers/<A>/<slug>` without naming the primitive): not judged.
  The text is untouched by this round's change set, which the fix-check rubric puts out of scope.

## Disputed

None.

## Deferred

None.
