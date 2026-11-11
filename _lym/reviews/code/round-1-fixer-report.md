# Fixer-report, code review round 1

## Changes

All three findings fixed as proposed, one commit each; `lym gate test ./internal/raddleparser ./internal/shedverbs` passes.

- F1 (`abded6ef`): `internal/raddleparser/check_test.go` gains `TestCheck_DependsOnNoteDetailNamesTheWayForward`, which asserts the Detail names the note, "drop the dependency" and the full `lym board promote '{"slug":"idea"}'` command.
  It also gains the row `DependsOnPartThatIsAlsoANoteIsNoNoteDependency`.
  That row expects `slug-taken` and no `depends-note`: a part slug that is also a board entry always trips `slug-taken`, so the row pins the part-over-note precedence through the absence of `depends-note`.
- F3 (`9e887b33`): `appendLoopChildOutput` in `internal/shedverbs/loop.go` returns early when the output, trailing newline trimmed, is empty.
- F2 (`8723ecaa`): `TestLoop_ChildStdoutIsMarkedInTheLoopLog` is table-driven, with the former case as a row plus rows for empty output, a trailing newline and a lone newline.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-affected-set: no findings.
- fix-proposal, LOW: `"\n\n"` still writes two bare prefixed lines. Disputed, premise false (see Disputed).

## Disputed

- fix-proposal LOW, premise false: the doc says "Empty output keeps nothing", and `"\n\n"` is not empty output.
  Its two blank lines are the child's own output, which the prefix marks faithfully, and `loopwait.go:253` drops them by prefix from the excerpt.

## Deferred

None.

## Rulings

None.
