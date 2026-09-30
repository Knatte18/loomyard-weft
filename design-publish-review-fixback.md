Every PR landed on 2026-09-30 needed a hub-side fix after review (false-pass paths in verify-triage, marker ordering in publish-verify-after-merge, pane injection gates in hub-orch-strand), done by a subagent outside loom.
Loom has no path back from Publish: the only verbs are approve and start.

Wanted:
- `lyx loom reject <review-file>` next to `approve`, recording the findings for the run.
- The next `loom start` routes the run from Publish back to Webster, which plans the findings as one or more fix batches through the normal fork/record/audit path.
- Integration verify (with triage) reruns, Publish updates the same PR, and the run awaits approval again.
- This is the role the recipe reserved for Tenter; decide whether it is Tenter or a smaller reroute.

Depends on webster-parent-merge-midrun because both change Webster run entry and batch state.