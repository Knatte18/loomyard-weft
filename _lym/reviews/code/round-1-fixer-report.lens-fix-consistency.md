```yaml
lens: fix-consistency
outcome: reported
```

Task: fix-consistency lens over the fixer's change set. One finding, plus a few items I checked that need no change.

**Findings**

1. **LOW** — `internal/verifytree/doc.go:97`, the package doc of the reach turn.
   - Summary: the package doc still says the turn "records the report's functions in the listed file", which reads as replacing the set. F1 changed the behaviour so a turn adds the pending functions to those already listed. The `WithReachTurn` doc comment in `reach.go` was updated to say that, but this twin was not.
   - Evidence: `doc.go:97` reads "The turn then records the report's functions in the listed file and returns one failing result marked WriterOnly." The updated sentence in `reach.go` reads "adds the pending functions to those the listed file holds".
   - Suggested edit: reword the `doc.go:97` sentence the same way.

2. **LOW** — `internal/testreach/doc.go:22`, the `ChangedLines` summary.
   - Summary: it says a line diff "adds or changes" lines without the new cap. F6 makes `ChangedLines` mark every head line of a rewritten stretch once its table would pass `maxLineDiffCells`. The function's own doc comment states the cap; the package doc does not.
   - Evidence: `doc.go:22` has the uncapped wording. I read only that one line, so I did not check whether a nearby line already covers the cap.
   - Whether to edit it is a judgement call: the summary stays true in general, but the same sentence is now partly stale.

**Checked, no finding**
- `internal/gatecli/doc.go` (grep hits at lines 470, 779 and others) mentions neither the root-commit skip F10 removed nor an index read lock for F11.
- `internal/testreach/doc.go:51` ("read whole, never indexed") concerns the worktree's own maps and is unaffected by F11.
- `internal/gatecli/reach.go` (F5): the new `reasonHeadUnreadable` follows the sibling reason constants in wording and in the way forward it names.
- `rowgate.go` (F4): the `WrapRowGate` doc says "The run after the fixer goes through the entry's re-run mark", and the live-record path now agrees. The `waitOnly` paths, used by other entries and by `Final`, still run without the mark. That matches the review's scope, which named only `spawning`.
- `fabricengine`'s `FirstParentCommit.Parent` doc ("empty for a root commit") stays true after F10; only the caller's skip was removed.

**Tool calls (1 of 6)**
- One `grep` over `testreach/doc.go`, `gatecli/doc.go`, the verifytree docs and the `.go` and `.md` files, for statements the fixes could have made stale. My context did not hold the docs outside the files that were read and changed.
