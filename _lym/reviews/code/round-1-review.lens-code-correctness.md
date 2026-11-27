```yaml
lens: code-correctness
outcome: reported
```

I ran the code-correctness lens over each function, branch and error path the diff adds or changes.

**What I ran, case by case:**

- **`fixerRun.run`, reading `TaskHead` at session start.**
  - Case: `TaskHead` fails. It returns `rec` unchanged before `chargeTurn`, so no turn is charged or written. Correct.
  - Case: the task head is the merge commit made before the session (the #271 incident, b6eb9). `startHead` is b6eb9. Correct.
- **`run`, ref deleted during the session.** Case: `remoteBefore` is "remote-1" and `remoteAfter` is "". `IsAncestor` is skipped, `predates` stays false, the end is pushed with `remoteTip` "", and the halt uses the delete wording. Correct.
- **`run`, ref moved to the start head.** Case: `remoteAfter == startHead` (b6eb9). `IsAncestor(b6eb9, b6eb9)` is true, so it logs Info and keeps the end the session decided (fixed in the incident). Correct.
- **`run`, first push during the session.** Case: `remoteBefore` is "" and `remoteAfter == startHead`, as when a background pusher first pushes the branch. `predates` is true, so no halt. Correct per the entry.
- **`run`, ref rewound to an older commit in the start head's history.** `predates` is true, so no halt and only a log line. Correct per the entry: the commit existed before the session.
- **`run`, tip that predates the session but is not in `startHead`, or a diverged tip.** `predates` is false, so it ends pushed. The wording "holds commits the task head … lacks" is true. The `startHead..remoteTip` range is git's set of commits reachable from the tip and not from the start head, which is what a human must check. Correct.
- **`run`, ref moved to a commit the session made (f1).** `IsAncestor(f1, startHead)` is false, so it ends pushed. Correct: the session could have added f1.
- **`run`, `IsAncestor` fails.** It returns an error with `session.rec`, whose charged turns are already on disk, and the decided outcome (possibly fixed) is dropped. This matches the existing `remoteAfter` read failure a few lines above, so the new path introduces no new kind of inconsistency. No finding.
- **The `IsAncestor` seam's arguments.** `(sha=remoteAfter, ref=startHead)` maps to `merge-base --is-ancestor remoteAfter startHead`, which asks whether `remoteAfter` is reachable from `startHead`. That is the right order. `startHead` is a full SHA from `HeadSHA`, so the leading "-" guard never trips. The closure opens fabric's warp checkout, which is the task worktree holding the remote-tracking ref's objects. Correct.
- **`sessionRef` cases:**
  - Done and cleaned: shuttle `finalize` removes the run dir before `Run` returns, so `Stat` fails and the result is "claude session &lt;id&gt;". Correct.
  - Timeout: `finalize` skips cleanup and `Stop` only records into the run dir (`stop.go:46-65`), so the dir exists and is named. Correct.
  - Done with KeepPane: the dir survives and is named. Correct.
  - Empty Engine: `TrimSpace` gives "session &lt;id&gt;". Correct.
  - Empty SessionID and RunDir: "none recorded". Correct.
  - `Stat` fails for a reason other than not-exist: the dir is omitted. Acceptable.
- **`haltReason`, pushed branches.**
  - Delete: names the ref, the row and the session, with its own way forward. Correct.
  - Move: names the ref, the row, the tip, the start head and the session, and the range in the way forward. Correct.
  - Neither branch says the session pushed. Correct.
- **`fixReviewOutcome.session`, `verdictSummary` and the `finalize.go` changed-branch halt.** They pass through `sessionRef` in the same way. Correct.
- **`recordLandedPush` trace line, by path:**
  - Default push (`branch=current`): ref is `ns.Branch(current)` and tip is the SHA resolved before the transfer. Correct.
  - Retry after rebase: a second `pushPlainWithRetry` re-resolves the SHA. Correct.
  - `PushCommit`: the tip is the given SHA. Correct.
  - `DeleteRemoteBranch`, `DeleteRemoteBranchLeased` and `UpdateRemoteBranchLeased`: tip is "" for a delete and the given SHA for a move. Correct.
  - A remote given as a URL is logged as that string. Acceptable.
  - Already up to date: still logs "push landed" at the unchanged tip. Harmless, since the logged tip equals the ref's current tip. No finding.
- **Fixture `IsAncestor` over a nil map:** answers false. The pushed row is `IsAncestor("remote-2", "head-1")`, false, so pushed. The no-push row hits the `"head-1..head-1"` key, true, so fixed. Correct.

**Findings:** none. No function, branch or error path the diff adds or changes gave a wrong result, a swallowed or misreported error, or a state left inconsistent beyond the existing pattern.

**Tool calls:**
1. grep of `shuttleengine/stop.go` for `RemoveAll`, plus `rundir.go:410-430`. My context did not show whether a stop or the orphan sweep removes the run dir before the timeout halt reason is built, which is what `sessionRef`'s "run dir" claim depends on.
