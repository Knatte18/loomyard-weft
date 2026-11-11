# Fabric way forwards name the anchored directory for every lym verb

In a hub whose anchor is a subdirectory, the FABRIC notice for a failed push said to run `lym fabric push` in the code worktree's root.
The cwd gate refuses every lym verb outside the anchored directory, so the command failed as written (issue #98).

## What changes

- `internal/fabricengine/failedpush.go` gains `lymVerbDir`, which reads the code worktree's recorded anchor through `lymcwd.NamedWorktree` and returns its anchored directory.
  It reads the marker when the line is built, spawns no git, and falls back to the root when no marker is recorded or one cannot be read (logged at Warn).
- Every way forward that names a `lym` verb in a code worktree now names that directory:
  - the retry push of a code or records failed-push record (`pushStep`), which the transport, auth, other and deferred classes and the records not-fast-forward step use;
  - the push after the merge of a diverged code branch (`codeDivergedWayForward`);
  - the push after accepting a rewind (`rewoundWayForward`), which said `then lym fabric push there` and now names the directory;
  - the push in the weft-replay conflict stop of `lym fabric pull` (`pull.go`);
  - the reconcile stops' `lym fabric pull` of a rewritten code branch, `lym fabric commit` of a dirty records branch, and `lym fabric anchor --set` of an anchor-stopped weft.
- A `git` step keeps naming the worktree root, where git runs from anywhere.
  The prime's code push stays a plain `git push` at the root.
- The `fabricengine` package doc states the rule beside the `PushFailedError` way-forward paragraph.

## Design decisions

- The anchored directory is resolved at render time rather than stored in the failed-push record.
  A record outlives an anchor change, and `FailedPushFields`, the notice and `PushFailedError.Error` all render through the same helpers, so one read covers every reader.
- A records-side `lym fabric pull` keeps its path: `lymVerbDir` applies only to a code worktree, since resolving a weft sibling's path would fall back to the hub-wide Bolt marker.
- The `lym fabric remove` path in the gone-branch way forward names the pair, not a cwd, and is unchanged.
- Out of scope: landingshed's push halt (`pushHaltReason`) still names the task worktree root.
  It is a Stuck reason rather than the FABRIC notice, and fixing it needs a new told `Deps` field wired from `internal/loomcli`.

## Tests

`TestFailedPushRecord`'s way-forward table gains rows over a temp hub whose task worktree records the anchor `src/app`:
a code transport retry, a records auth retry pushing in the code worktree's anchored directory, a code not-fast-forward merging at the root and pushing in the anchored directory, and a rewind resetting at the root and pushing in the anchored directory.
The existing rows, over paths with no marker, keep naming the root.
