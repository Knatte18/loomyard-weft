# Landing: verify the tree after merging the parent

`Publish` merged the parent into the task branch and pushed without compiling or testing the result.
A merge can build cleanly and still break tests; in run `operator-surface` a conflict-resolved merge left four test call sites on an old signature, and only the PR review caught it.

Now every parent merge-in that changed the task tree is followed by the plan's `## verify:` command, in the task worktree root.
A failure returns Stuck before anything leaves the worktree.
`Publish` makes no push and no GitHub call; `Finalize` does no parent-side merge, no `MarkTaskDone`, no parent push and no PR close.
The Stuck reason names the parent branch, the exit code (or spawn error) and the path of the output file holding the command's combined stdout and stderr.

The board entry links no GitHub issue, so there is no `Closes` line.

## What changes

- `internal/landingshed`: a verify gate (`verifygate.go`) runs in `Publish` between the merge-in and the push, and in `Finalize`'s merge-in step, so every retry after a merge-in-required refusal is gated too.
- `internal/verifyrun` (new): the in-process shell runner (`sh -c`, `cmd /C` on Windows) with a context and an output writer.
  Webster's integration bisect now calls it with a background context and a discarding writer, so its behaviour and log lines stay as before.
- `internal/loomengine`: `LoomVerifyPendingPath` and `LoomVerifyOutputPath`, both under loom's scratch directory.
- `internal/loomcli` (`landingdeps.go`): fills the new `Deps` fields; the drift guard covers them.
- Docs: `landingshed` package doc, the `## verify model` section of `contracts/specs/loom-plan-spec.md` (landing is the verify line's second consumer), and `docs/overview.md` for the new package.

## Design decisions

- **Trigger:** the gate runs when `mergeresolve.Result.AlreadyUpToDate` is false, whether the merge was clean or resolved in a conflict session, because a clean textual merge can still break compilation.
  A stuck resolver outcome returns before the gate, as before.
- **`Finalize` is gated too**, although the board entry names only `Publish`.
  It has the same failure mode and lands on the parent with no reviewer downstream.
- **Command source:** a closure in `Deps` that `internal/loomcli` fills, parsing the plan through `internal/planparser` each time the gate runs.
  A value would be empty at bootstrap on a fresh run, before the plan exists, and the gate would silently skip.
  A plan read or parse error is a returned error, never a skip.
- **No `## verify:` section means no gate:** a `logger.Warn` names the skip (and the pending marker, if present), and the producer proceeds.
  There is no language-specific fallback command.
- **Pending-verify marker:** written after a tree-changing merge-in and before the command, removed only on a pass.
  If it exists when the producer reaches the gate, the gate runs even on a no-op merge-in, so a resume after a failed verify cannot push the unfixed tree.
  One marker is shared by both producers.
- **The merge commit stays on failure**; the operator fixes forward on the merged tree rather than losing a conflict session's work.
- **Error classes:** marker-write or output-file-create failures and plan errors are returned errors, not Stuck.
  A cancelled context returns the cancellation error and leaves the marker.
- **Told paths:** landingshed derives no path; the marker and output paths come from `loomengine` through `Deps`.
  No recipe row changes, and neither producer gains an `on_stuck`.

## Known limits

- A plan whose `## verify:` names only the task's own packages can miss a break in a parent-side caller; `./...` closes that, and this change does not force it.
- A process killed between the merge-in and the marker write leaves an unverified merge with no marker.
- Cancellation kills the shell, not commands a chained verify line already started; those may keep writing to the output file.
- A slow plan-level suite now runs once more per landing whose parent moved, and again on each `Finalize` retry.

## To check by hand

- Run `Publish` against a real parent that moved with a semantic conflict, and confirm the Stuck reason, the output file and the surviving marker.
- Confirm on Windows that `cmd /C` runs a chained `&&` verify line the way `sh -c` does; only POSIX was exercised here.
