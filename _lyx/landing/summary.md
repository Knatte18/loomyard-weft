# Darn verify failures route to Merge-In when Darn-Write cannot own them

A verify failure in code the task's diff cannot reach, or one the parent has since fixed, no longer loops through Darn-Write.
The same change keeps an exhausted gate's findings file alive until its halt is read, and stops the impacted set from adding binary-driving test packages to diffs that only reach them through the `lyx` binary.

Closes #714
Closes #765
Closes #785

## What changes

- **Verify-failure routing (darn recipe, Merge-In and Publish).**
  A failure is classified as owned or not owned by the task, against the impacted set `internal/impactset` derives from the merge base to the working tree.
  An owned failure routes to `Darn-Write` with today's way forward.
  A not-owned failure routes to `Merge-In`, and its Stuck reason names the failing packages, says they lie outside the task's impacted set, and gives the way forward: fix the parent, then `lyx loom goto --to Merge-In` and `lyx loom resume`, or `goto --to Darn-Write` if the parent judges the task caused it.
  The existing single automatic retry now re-enters Merge-In, which merges a moved parent and re-runs the failed gate, so a flaky test also gets one more run.
  The loom recipe keeps its fixed `Webster-Burler-Review` target.
- **Darn-Write entry reroute.**
  Before spawning the writer, Darn-Write returns a reroute to Merge-In when this run's failure record is present, HEAD is still the record's `head`, the task worktree is clean, and a parent tip Merge-In would merge is not an ancestor of HEAD.
  No writer is spawned and no bounce budget is spent.
  The gate-exhausted Darn-Write halt's way forward now names both the re-spawn and the Merge-In reroute.
- **Gate findings file.**
  An exhausted gate writes its last failing findings to a per-row file under the run's shed run directory (declared in `internal/shedrun`), and the Stuck reason names that file.
  It survives `lyx loom goto`, `lyx loom resume` and a pause, and the row's next call whose gates pass removes it.
  `internal/shuttleengine` hands the findings text out on its gate outcome, since finalize removes the shuttle run directory first.
- **Impacted set.**
  Binary-driving test packages (those importing `internal/testkit/lyxbin`) now join the set only when the diff changes the `cmd/lyx` main package itself.
  Otherwise they join through the import graph.
  A change confined to `internal/planglyph` no longer runs `internal/reedcli`.
  This applies to every consumer: darn's gates, loom's round gate and computed-scope `lyx gate test`.
- **Shell guard.**
  `TestRows_PatternsDenyAndAllow` pins that `lyx gate test --wait --tier 2 <pkg> -- -run '^TestX$'` passes every row; the rows are unchanged.
  `TwoWorktreesBootedConcurrently` (#714) was already fixed by `98b171b05`.

## Design decisions to review

- The classifier runs in the CLI wiring and is told to `internal/landingshed` as a seam; a nil classifier keeps the fixed values the loom recipe uses.
- Every uncertain case counts as owned: a Whole fallback, a derivation error, an opaque-only failure, or a package-level or build failure in an impacted package.
  Only a failure whose packages the import graph places outside the diff's reach is not owned.
- The classifier changes only the record's target and the reason's way-forward text; every failure still halts the row, and no retry is added.
- The reroute reads tips with no fetch (`fabric-remote-async`), so a parent fix seen only on origin is missed until something fetches.
- Only Merge-In merges the parent (`PATTERN-parent-lock`); Darn-Write reroutes instead of merging.
- The reroute skips the Darn-Review perch and Describe; the HEAD and clean-tree conditions make that safe, since no code has changed since the failed verify except the parent merge.
- A binary-driving test that reaches a change only through a subcommand its own package does not import leaves the narrowed run; the whole-tree runs at Merge-In, Publish and Finalize still run it, and the classifier counts such a failure as not owned.

## Check by hand

Nothing here was exercised against a real parent move or a real tmux-tier failure.

- A reroute whose Merge-In verify still fails (an owned failure the parent's change did not cure) finds the one retry spent and halts for the parent instead of reaching the writer.
- An owned failure the parent fixes after the writer committed, or while it left uncommitted edits, is not rerouted; the writer loops through its gate budget and halts at Darn-Write.
  `gotoAdmitted` in `internal/shedengine/goto.go` admits only Preflight and Darn-Write from that halt, so the recovery drops the writer's work: reset to the record's `head`, discard uncommitted changes, then `lyx loom goto --to Darn-Write` and `lyx loom resume`.
- An unparsed failure beside a parsed failure outside the impacted set classifies as not owned, because the parser emits no opaque identity once it finds another; the hand-back reason then wrongly places the failure outside the task.
- A deterministic not-owned failure on an unmoved parent spends one futile gate re-run before the hand-back.
- Run `lyx gate test --tier 0 --tier 1 --tier 2` on `main` after landing, as CLAUDE.md requires before a push.
