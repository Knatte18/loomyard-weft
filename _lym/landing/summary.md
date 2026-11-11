# beam merge-parent skips slugs a run directory or task branch holds

`lym beam merge-parent <beam>` now picks a merge task slug that no board entry, batten run directory or task branch holds.
Before, it checked the board only.
A beam that had merged its parent once, through a completed and torn-down `<beam>-merge-in` run, got a second task under the same slug.
Dobby refused to start that task, because the completed run's directory and the torn-down pair's branch outlive the board entry.
The orch had to rename the task by hand.
Now the second merge is filed as `<beam>-merge-in-2`, and the next as `-3`, and dobby starts it.

Closes #115

## Design decisions

- `shedrun.RunIDs` lists the durable run directories under an anchor, in name order, and lists none when the shed directory is absent.
  It replaces the directory scan that `adoptcli`'s `hubFixDeps` carried inline, so both callers read the run directories one way.
- `landDeps` gains a `HeldSlugs` seam.
  `hubLandDeps` fills it with the run directory names from `shedrun.RunIDs` and the task branch slugs from `fabricengine.TaskBranchSlugs` with an empty prefix.
  `merge-parent` is its only caller.
- `freeMergeTaskSlug` treats a held slug like a board entry, so the numbering and the cut to the board's slug length are unchanged.
- `merge-parent` refuses when `HeldSlugs` fails, naming the cause and the command to run again.
  Filing under a slug it could not check would bring back the refusal at start.
- The `merge-parent` help text and the `beamcli` package doc describe the new rule.

## To check

- `hubLandDeps` wires `HeldSlugs`, and `TestHubLandDeps_HeldSlugs` in `internal/beamcli/deps_integration_test.go` covers it over a real hub pair.
  It is a tier-1 integration test and was not run by hand.
  Check that it passes and that its run-directory and task-branch cases fail without the wiring.
