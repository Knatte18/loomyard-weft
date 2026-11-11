# Push and reconcile way forwards name the anchored directory

In a hub anchored at a subdirectory, the failed-push notice told the orch to run `lym fabric push` in the worktree root (issue #98).
Every fabric verb refuses that cwd, because `lymcwd.Resolve` requires it to equal the worktree's anchored directory.
The command in the notice therefore failed as written.

## What changes

- `FailedPush` gains `FabricDir` (`fabric_dir`): the anchored directory of the pair's code worktree.
  `pushJob.record` fills it for code and records records from the worktree's own recorded anchor, through `lymcwd.NamedWorktree`.
  An anchor that cannot be read is logged and leaves the root.
- `pushStep`, `codeDivergedWayForward` and `rewoundWayForward` name `lym fabric push` in that directory.
  Each git command they name, such as `git merge`, `git reset --keep` or `git push`, still runs in the branch's worktree.
  The rewound line now names the directory instead of "there", because it can differ from the reset's worktree.
- The prime's code branch still names `git push` in the prime worktree.
  A bolt record still names `lym board sync`.
  A gone record still names the pair at its root, since `lym fabric remove` runs from the prime.
- The origin reconcile carries the same directory as `BranchOutcome.FabricDir`.
  Its rewound, rewritten, diverged and dirty stop lines name `lym fabric push`, `lym fabric pull` and `lym fabric commit` there.
- The pull's weft-conflict stop and its rewound refusal name the warp location's `AnchorPath()` for `lym fabric push`.
- The `fabricengine` package doc says where each fabric verb and git command a way forward names runs.

## Design decisions

- The directory is stored on the record when the record is written, not resolved when the line is rendered.
  `FailedPushWayForward` stays a pure function of the record, and `lym fabric status` shows the same line the mail carried.
- A record written before this change has no `fabric_dir`, and it reads as anchored at the code worktree's root.
  That reproduces the old line until the next failure of the branch rewrites the record.
- The anchor-stop way forward, `lym fabric anchor --set` in the code worktree, is unchanged.
  It is the step that changes the anchor, so it is outside this fix.

## Tests

- `TestFailedPushRecord` gains anchored rows for code transport, records auth, code not-fast-forward and code rewound.
  The existing rows, which have no `FabricDir`, pin the fallback to the root.
- `TestPushFailureClasses` gains `APairPushFailingInASubpathAnchoredHubNamesTheAnchoredDirectory`.
  It refuses a pair's code or records push in a hub anchored at `backend`.
  It checks that the record's `FabricDir` and the mail's way forward name `<pair>/backend`.
- The reconcile and pull integration tests pass the directory to the changed helper signatures.
  Their hubs are anchored at the root, so it equals the worktree.
