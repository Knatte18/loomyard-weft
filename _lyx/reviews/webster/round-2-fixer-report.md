---
fixed: [F1, F2, F3, F4, F5]
deferred: []
---

All five findings of `round-2-review.md` are fixed, each in its own commit on the task branch, none pushed.

## Changes

- **F1** (`dd9e54674`): broke each flagged comment at its independent-clause boundary in `shuttleengine/run.go`, `doc.go`, `engine.go`, `spec.go`, `fakes_test.go` and `startup_test.go`; the `skillReed` comment's first clause became its own sentence.
- **F2** (`899c0ac03`): the same in `claudeengine/skillload.go` (header, `ClassifySkillLoad`, `classifyWindow`) and `claudeengine/doc.go`.
- **F3** (`4e26093b3`): the same in `orchengine/doc.go`'s reload-sequence section, in the two loomcli stub-script comments, and in `watcher_test.go`'s `timeout` field comment, which now reads as one clause.
- **F4** (`21513d3ed`): added a row to `TestWatcher_SkillSkipCauses` where the skills turn reports a missing skill and the retry turn never ends.
  It asserts that only the retry skill is skipped with `cause=timeout`, that no second retry is typed and that the pointer follows.
  The table now reads its turn ends before the timeout, which leaves the existing timeout row unchanged.
- **F5** (`d14c66aa4`): the reset assertion compares `ReloadStep` with `ReloadStepSkills`.

## Verification

`go build ./...` and `go test` under no tag, `integration` and `tmux` pass for the plan's verify packages, and `go vet -tags llm` is clean.
The new `TestWatcher_SkillSkipCauses` row passes.

## Prior rounds

Round 1's fixes held: the reload tests still run in parallel, the empty-`reload_retry` row and the log assertions are in place, and production code assigns only the named `ReloadStep` constants.
Round 1's F1 recurred in a wider set of places because its fix covered the flagged lines, not every comment line the diff adds or changes.
A comment-line-break check over the whole diff (comment lines containing a comma before `and`, `so` or `but`, or a semicolon) is the cheap way to close that gap for the next author.

## Deferred

Nothing deferred.
