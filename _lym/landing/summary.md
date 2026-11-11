# beam merge-parent skips slugs a completed run or leftover branch holds

`lym beam merge-parent <beam>` picked its task slug, `<beam>-merge-in` numbered from `-2`, by checking only the board's entries.
A completed merge-in run outlives its board entry: its batten run directory stays on the prime until `lym batten cleanup`, and its torn-down pair leaves its task branch behind.
A second merge of the parent therefore got the old slug again, and dobby refused to start it as already completed, so the beam orch had to rename the task by hand (issue #115).

## Change

- `freeMergeTaskSlug` in `internal/beamcli/mergeparent.go` now skips every slug that a board entry, a batten run directory or a task branch holds.
  A task branch counts as local or as an origin remote-tracking branch.
- `landDeps` gains a `HeldSlugs` seam.
  Over a real hub it lists the prime's run directories and every task branch slug through `fabricengine.TaskBranchSlugs`, read in-process with no network call.
  A failed read refuses the filing, files nothing, and names the re-run as the way forward.
- `shedrun.RunIDs` lists the durable run directories under an anchor.
  Adopt's fix loop had the same listing inline and now calls it, since both callers pick a slug no run directory holds.
- The `beamcli` package doc and the `merge-parent` help text name the new rule.

## Design decisions

- Any run directory holds its slug, whatever the run's state, not only a done one.
  A run directory in any state makes a start of that slug resume or refuse instead of starting fresh, so a new merge task must not take it.
- Task branches are listed with an empty prefix rather than the beam's slug.
  The beam's slug is cut by a different length for each numbered candidate, so no single prefix covers them all, and a local ref listing is cheap.

## Tests

- `TestMergeParent` gains a row where a completed run holds `b-merge-in` and a leftover branch holds `b-merge-in-2`, so `b-merge-in-3` is filed.
  A second row covers the refusal when the held slugs cannot be read.
- `TestRunIDs` pins the directory listing: name order, plain files skipped, and an absent shed directory listing none.
