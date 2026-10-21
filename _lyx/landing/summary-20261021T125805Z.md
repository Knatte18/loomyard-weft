# Origin reconcile before acting, and push failures never silent

A machine no longer acts on a stale view of origin, and a push origin refuses is no longer silent.
Before each step where a stale view does harm, one origin reconcile compares this machine's branches and Bolt entries with origin.
It fast-forwards, cleans up a run that landed elsewhere, or stops naming the diverged branch.
A rejected push now retries where that is safe, and otherwise writes a failed-push record and mails the orch.

Closes #737

## What changes

- `internal/fabricengine` gains the origin reconcile: two parallel `git ls-remote origin` calls (warp repo, records repo), one fetch per repo of only the moved heads, and one typed report with an outcome per branch and per entry.
  Branch outcomes are level, fast-forward, ahead, rewound, rewritten, diverged, gone, plus busy, live and dirty where a writer or a tracked change blocks a move.
  Entry outcomes are `active`, `ended` and `stopped`, from one classifier.
- Call sites: `lyx orch start`, the hibernate wake, batten resume, Merge-In, Finalize and `lyx cleanup`.
  `lyx orch start` never refuses; the run-scope sites refuse or Stuck on `ended` and `stopped`.
  `lyx cleanup` without `--apply` judges only and moves no ref, tracking refs included.
- A records replay replays local commits onto origin, drops those that become empty and stops on a real conflict.
  A weft replays by cherry-pick; Bolt replays by entry through `internal/boardengine`, keeps origin's value on a same-field conflict, and keeps the lost local commit under a local `set-aside/<slug>/<sha>` tag.
  The reconcile, `Fabric.Pull`, the push retry and Bolt's pull-first share it.
- `lyx fabric pull` fails with a named way forward when the weft cannot be brought level, instead of `ok: true` with `weft_pulled: false`.
  Its warp arm judges unpushed commits and a rewind by the tracking ref's reflog.
- A push classified as not-fast-forward on the records side fetches, replays and retries once; a held write lock defers it.
  Auth refusals, a second rejection and a failed replay write a failed-push record under the hub's `.lyx` directory and send one creel mail signed `FABRIC`, or by the calling strand in an agent pane.
  `lyx cleanup`, `lyx fabric status` and `lyx tender status` print outstanding records, and `lyx cleanup --apply` no longer reports a spawned push as landed.
- `internal/pairteardown` gets a local mode that runs no origin step, tags the weft tip locally instead of pushing an archive tag, and waits only for its own loop kill.
- The Preflight row's records push is now detached.
- `PATTERN-origin-reconcile` and `PATTERN-push-failure-visible` are added; package docs and `docs/overview.md` are updated.

## Design decisions to know

- The reconcile judges and moves only forward: it never resets away a commit origin lacks, force-pushes, rebases a code branch or recreates a remote branch.
- An entry is ended only when origin's Bolt tip and its history show it `done` or removed, and this machine holds a batten run record of that slug; an entry only local Bolt holds is never ended, and a run whose local history shows Finalize entered is never ended.
- The prime's branches move only under a try-acquired prime lock with no live parent-lock owner; every lock the reconcile needs is one try-acquire with no wait, and a held lock reports busy.
- A task branch rewound on origin stays reported rewound until the operator settles it.
  The keep-local way forward is `git push origin <branch>` in the task worktree, not an `ours` merge, which would change nothing.
  `lyx fabric pull` refuses a rewound task branch and never settles it itself.
- Wake retries an unreachable origin every 10 seconds for 2 minutes, then moves nothing and names `lyx cleanup --apply` in the notice.
  Wake passes the network gate only for its own pause entry; another hub entry skips the reconcile.
- An unreachable origin is a warning and every call site carries on.
- `battencli` and `battenshed` reach a records stop through `RecordsStopMessage` and `NewRecordsStopError`, to stay within `PATTERN-fabric-vocabulary`.
- `Landing.Reconcile` is nil-legal in the seam fill checks.

## Check by hand

Not run by hand: the real two-machine and hibernate-wake paths, and a real 403 push against GitHub.
Run `lyx gate test --tags integration ./...` before pushing to `main`.

Review findings left open, to check in the code:

- `pattern/`: entries rewritten by the change are not all declared in the change's PATTERN changes.
- `internal/fabricengine`: the reconcile's verdict for an entry applies no slug filter.
- `internal/fabricengine` replay: set-aside tags are written after the reset, not before.
- `internal/fabricengine` push classification: an unclassified push failure is not recorded.
- `internal/orchcli/start.go`: an error from the reconcile reaches `lyx orch start`'s envelope bare.
- Merge-In and Finalize in `internal/landingshed`: Publish's not-fast-forward path skips the class line.
- `internal/fabricengine` and `internal/pairteardown`: a created set-aside tag has no mutation kind.
- Way-forward lines: their text is not asserted at every print hop, carries no directory, and shares one re-entry verb string.
