```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of decisions that act on the same state, file, ordering or invariant.

**Findings**

- **MEDIUM — D2: the copy step is not ordered with the live record's lock.**
  D2 says each change to the ask record "is followed by a copy", and that the last copy is the final state.
  The live record changes under `state.UpdateJSON` with its lock (`burlerengine/parentreview.go`), but nothing makes the copy happen under that lock.
  Two changes close together, for example an answer racing an expiry, or `SettleParentAsk` racing `MarkParentAskPosted`, can write their copies out of order.
  The tracked copy then holds the earlier state.
  That breaks D2's claim and the D2 criterion that the copy matches the final state.
  Fix: write the copy inside the same locked update, or read the record again under its lock just before copying.
- **MEDIUM — D5 with D4: extracting an older run has no defined file set.**
  `--extract` writes "the kept records", and D4's index row is the only list of them.
  A run archived before D4 has no row, but D5 promises it a derived row, and that row's set of kept records is never defined.
  For example, does it include the legacy `parent-review/round-*` directories, `_lym/plan/` or `_lym/usage/`?
  The D5 extract criterion covers only a run that has an index row.
  Fix: define the derived set, for example fixed subtrees of the tag such as `_lym/discussion/`, `_lym/reviews/`, `_lym/landing/` and `_lym/drift/`, and add a criterion that extracts an older run.
- **LOW — D3 with D4: a fixer that runs after the index row is written is missing from the row.**
  D4 writes the row "once Finalize has landed the code".
  A landing fixer that runs later in Finalize, such as one after a post-merge test failure, leaves a tracked report that the row does not list.
  D3 still keeps the file, so only the index is incomplete.
  Fix: write the row as Finalize's last step, or rewrite it after each later fixer ends.
- **LOW — D2 with D4: an ask copy changed after the row is written makes the row stale.**
  The row is written at landing, but an expiry or clear that runs later in Finalize rewrites the ask copy.
  The path stays the same, so the row is not wrong about where the file is.
  Its content just postdates the row, so this does no harm unless the row comes to hold ask outcomes.
- **LOW — D5 with D4: "newest run" is not defined when a slug has several archive tags.**
  The tag name carries only a SHA prefix.
  The record does not say whether "newest" means the index row's landing time, the tag's creation time or the tip's commit time.
  An older run has no index row to sort by.
  Fix: name the order, for example the tip commit time, which every tag has.
- **LOW — D1: a send-back and a fix-here on the same round may number their copies the same.**
  D1 numbers copies "by send-back or fix-here round".
  Today send-backs carry a number `n` (`pr-findings-<n>`), while a fix-here answer keeps only a count, so the two series can share a number in the same directory.
  The D1 criterion checks only that a second round does not overwrite the first, not that a send-back and a fix-here do not collide.
  Fix: one counter for both, or a distinct prefix for each kind.

**Pairs checked**

- D1 and D2, the reviews tree and PR-Review ask answers alongside PR findings: held. The directories differ and both are tracked.
- D1 and D4, the index listing the PR findings paths: held. PR-Review runs before Finalize writes the row.
- D2 and D4, the ask copy against the index: held for paths, stale content possible (LOW above).
- D2 with itself, a copy against the live lock under concurrent updates: broken (MEDIUM above).
- D2 and D5, the new ask directory against the legacy `parent-review` decoder: held. The record keeps the directories distinct and a criterion covers it.
- D3 and D4, landing fixer reports against when the row is written: held for fixers before the merge, partial for fixers after it (LOW above).
- D1, D2 and D3 against teardown's commit of uncommitted records: held. The research found that `Remove` commits the sibling worktree's records before tagging.
- D4 and D5, the row against the derived row: held for listing, undefined for extract (MEDIUM above).
- D4 on restart, a resumed Finalize with the same landing commit: held. The rewrite gives the same content.
- D5 on its own, "newest" among several tags: underspecified (LOW above).
- D5 and D6, read-only report against tags never deleted: held.
- D6 against the existing `PlainArchiveTagsOnOrigin` cleanup: held. The exception is stated.
- D1 with itself, send-back and fix-here numbering: possible collision (LOW above).
- The constraint "no new push or ref write" against D1 to D5: held. D5 writes only under `<dir>`.
- The PATTERN change text against D1 to D4 and D6: held. Its terms match the decisions.
- Each D1 to D6 criterion against its decision: held, apart from the gaps in D2's copy ordering and D5's older-run extract named above.

Tool calls made: none.
