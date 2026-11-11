```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: checking that every bound, behaviour or code fact the fix added or rewrote holds against the code at HEAD.

**MEDIUM: the claim that the fresh Merriam session carries the replaced record is false.**
- **Changed text:** D1 notice bullets: "The reason tells the parent to re-enter the run's Webster, so the fresh session carries the replaced record". The D1 criterion naming re-entry into Webster rests on the same claim.
- **Evidence:** a fresh master prompt reads only the record's `Added after Discussion` entries and its declared PATTERN changes.
  - `internal/websterengine/runlevel.go:734` calls `discussionparser.AddedDecisionTexts(deps.Geom.DecisionRecordPath)`.
  - `internal/websterengine/doc.go:340` says Run "renders every `Added after Discussion` entry of the decision record verbatim into the master prompt".
- **Why it matters:** a replace folds added entries into the main decisions, so a re-entered Merriam gets less of the folded design than before, not the replaced record.

**MEDIUM: "the only liveness read" is false.**
- **Changed text:** D4 bullet: "`internal/boardcli` imports nothing from batten, and the only liveness read, `registeredBattenAlive` in `internal/pairteardown`, reads this machine's process records."
- **Evidence:** `internal/loomcli/driverprompt.go:100-110` reads batten liveness too. `driverWatched` and `liveBattenPID` read a batten-watched marker file and check its pid. The review itself cited `driverprompt.go:107` as a second liveness read.

**LOW: "tags origin's tip" misstates what the teardown tags.**
- **Changed text:** D4 bullet: "That teardown deletes the local weft branch, tags origin's tip under `archive/&lt;slug&gt;/` and deletes origin's branch".
- **Evidence:** the `fabricengine.Topology.Remove` doc (`internal/fabricengine/remove.go:165-183`) says the tag goes on the local weft tip.
  - The teardown first pushes the local tip to origin when origin lacks it.
  - A second tag goes on origin's tip only when that tip is neither the local tip nor an ancestor of it.
- **Effect on the conclusion:** none. The pushed tag covers origin's tip in the usual case, so the branch is still not live.

Checked and holding:
- `teardownRequest` (`internal/battencli/wire.go:447`) and cancel (`internal/battencli/cancel.go:717`) both pass `Remote: true`.
- `raddleRunFiling` reports the filed slugs.
- `CommitEmpty` refuses while the index holds staged content.
- `StageAndCommit` returns committed false with nothing staged.
- Record-Write never overwrites an existing record file.
- `convertRunEntry` writes the project's own record.
- `FindTrailerCommit` matches on the key alone.
- `IdentityText` strips added entries and carry-over blocks.

Tool calls:
1. Read `internal/battencli/wire.go:436-448`, `internal/loomcli/driverprompt.go:98-115` and `internal/loomcli/raddlewiring.go:66-95`, to check the finish request, the second liveness read and what `raddleRunFiling` reports. My context had none of them.
2. Grepped `internal/websterengine` for decision-record reads, to check what a fresh Webster session's prompt carries. My context lacked that code.
