# Fabric teardowns push and delete on origin async; local-only pairs go

`lym fabric remove`, the task-branch discard and `lym fabric cleanup --apply` no longer wait on the network.
They run their local steps and return, and a detached child pushes the archive tags and deletes the branches on origin, in the order the teardown lock guards.
The local-only pair mode (`lym fabric add --local`) is removed: no flow uses it, and a batch pair's commits belong on origin like any pair's.

## What changes

- **Local-only pairs removed.**
  `lym fabric add --local` and `Topology.AddLocal` are gone, along with gitrepo's per-branch `lymlocal` mark and the push refusal it drove (`internal/gitrepo/branchmark.go`).
  Also gone: the `LocalSkipped` reporting in pushes and spawns, the local paths of `Remove` and `Cleanup`, the remote-gate refusal of a marked branch, and the launcher-restore skip.
  The tests and the sandbox scenario F23 that covered them go too.
  `RemoveLocal`, the no-origin removal of an ended pair, is a different mode and stays.
- **Teardown remote steps in a detached child** (`internal/fabricengine/remoteteardown.go`).
  - **Remove:** `Remove` tags the local weft tip `archive/<slug>/<tip>` before its first teardown mutation (`tagWeftTipForArchive`), then removes launchers, worktrees and local branches as before.
    The child then pushes the weft tip when origin lacks it, then the archive tag (plus a second tag at a diverged origin tip).
    With `--remote` it also makes the leased weft delete and then the landed task-branch delete on origin.
  - **Discard:** `DiscardTaskBranch` tags the local task tip and deletes the local branch under that tag; the child reads origin's tip, pushes the covering tags and makes the leased origin delete.
  - **Cleanup:** the cleanup sweep tags and deletes each orphan locally; its child pushes the tags and, with `--remote`, deletes the branch on origin.
  - **Lock and handover:** the child is the hidden verb `lym fabric remote-teardown`, started in the Bolt checkout and registered in runproc under the slug.
    It takes the slug's teardown lock exclusive before the command's first mutation and answers `held` or `busy` on one line.
    It then reads a JSON job from stdin and keeps the lock until its last remote step, so the weft tip → archive tag → leased delete order of `PATTERN-slug-remote-order` holds while the command returns at once.
    A command that fails closes the child's input with no job and waits for it to let go of the lock before it respawns the pair's push.
  - **Failures:** a failed remote step is logged and mailed to the hub's orch through the installed push notifier.
    The mail is one line naming the slug, the operation and the way forward: the `git -C … push origin refs/tags/archive/…` that pushes the kept local tag, and a re-run of `lym fabric remove <slug> --remote`.
  - **Result:** `RemoveResult.RemoteHandedOff` (envelope `remote_handed_off`) says the remote steps went to the child.
- **Parallel add probes.**
  `fabric add` reads the warp branch's origin tip beside the weft probe (`judgeWarpLeftover` judges it once the weft answer says whether the pair is live).
  The add now waits on one round of network instead of two; these probes stay synchronous by decision.
- **Support:** `gitrepo.Repo.PushCommit` pushes a SHA to a branch without needing a local branch, which the child uses once the local weft branch is gone.
  `archiveWeftTip` takes the tip the command read.
- **Docs:** `PATTERN-slug-remote-order` notes that a teardown's exclusive lock is held by its child until the last remote step.
  `internal/fabricengine/doc.go`, `internal/fabriccli/doc.go` and `docs/overview.md` follow both changes.

Every other remote call stays synchronous, because the command reads its result before it goes on.
That covers origin reconcile, the pull, merge, checkout and reconcile fetches, the landing pushes, the GitHub calls, a direct `lym fabric push`, clone and the Bolt push.
The parent recorded that scope in the decision record (`Added after Discussion`, 2026-11-03).

## Design decisions to review

- **Lock handover by a child that takes the lock first.**
  gofrs/flock (v0.8.1) gives no access to its file handle, so the lock cannot pass to a child as an inherited descriptor.
  Instead the child takes the lock before the command's first mutation, and the command does its local work while the child holds it.
  Releasing and re-taking the lock would open a gap in which an `Add` of the slug could adopt the origin branch the child is about to delete.
- **The archive is no longer the gate before teardown.**
  Before, a failed tag push left the whole pair in place.
  Now the archive tag is made locally before any deletion, and the deletions only need that local tag (the archived-tip gate reads local refs).
  A failed push therefore leaves the pair removed and the tip held by the local tag, with the way forward mailed to the orch.
  The remove, discard, cleanup and batten tests that inject an unreachable origin were updated to this contract.
  A `lym fabric cleanup --apply --remote` whose archive fails now reports `"partial": true`, since the local delete precedes it.
- **Test binaries run the job in process.**
  Under `testing.Testing()` the hold stays in process and the job runs before the verb returns, filling `OriginArchiveTag`, the `RemoteBranch*` and `RemoteCodeBranch*` fields, the sweep entry's remote fields and `DiscardResult.ArchiveTags`.
  This mirrors `SpawnDetachedPush`'s refusal to fork under test, and it keeps the existing remote-outcome tests meaningful.
  In production those fields stay empty and `RemoteHandedOff` is set; `fabriccli`'s non-zero exit on `RemoteBranchError` and batten's halt on it now only fire for an in-process run.
- **Role guard.**
  The teardown child inherits the strand name the way the push child does, so `sandboxedRoleRefusal` lets `remote-teardown` through.
  The guard stops mistaken calls, not deliberate ones, exactly as for the push child's bypass flags.
- **One child per orphan in a cleanup sweep**, each taking its slug's lock by one try; orphans are rare, so no sweep-wide child was built.

## Check by hand

- No test was run in this session; the verify gate runs the suite on the committed tree.
- Run `lym fabric remove <slug> --remote` on a live hub.
  Check that it returns with `remote_handed_off: true`, that the archive tag and branch deletes land on origin shortly after, and that a `lym fabric add <slug>` started meanwhile waits for the child's lock.
- Break the records origin and remove a pair; check that the orch receives the one-line notice and that its `git push` way forward works.
- The child's stdin/stdout pipes with `proc.Detach` were not exercised on Windows.
