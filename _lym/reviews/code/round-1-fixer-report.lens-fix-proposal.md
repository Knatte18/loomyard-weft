```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the proposals the fixer carried out for F1–F5. F4 and F5 are comment-only marks, with no case against them.

**F2 — MEDIUM** — `internal/beamcli/deps.go` `PushBeam`, now `pairteardown.PushRun(prime, beam, slug, RunPush{LockWait: ...})`.
- The reviewer offered either `PairPush` with `Here` and `RunSlug`, or `PushRun`. `PushRun` also runs `HandOver` on a takeover error, and `HandOver` has side effects `PairPush` alone lacks.
- With `InsidePair` false, `HandOver` ends the subject processes for `slug` and calls `td.RemovePair(Request{Slug, Local: true, Force: true, ...})` (`internal/pairteardown/handover.go:111-117`). That would be a forced removal of the beam's pair from inside a beam verb.
- The other `PushRun` callers (`loomcli/wiring.go:225`, `landingdeps.go:416`) pass `InsidePair: true`. `beamcli` is the only caller that lets the pair be removed.
- `HandOver` also reads the batten record and `OriginBattenStatus` for the beam's slug and resolves `td.taskLocation(slug)`. The beam is not a batten run, and that mapping is untested for a beam.
- `pr.go:81-83` wraps any `PushBeam` error, `ErrHandedOver` included, as `rerunRefusal(... "push the beam branch" ...)`. A handed-over run, with its pair already removed, is then reported as a push failure to re-run.
- The fix added no beamcli test, so none of this is exercised.

**F1 — LOW** — `pushOwedCodeTip`, the `originAbsent` branch, which does a plain `repo.PushCommit(originRemoteName, tip, branch)`.
- The reviewer asked for a lease that expects the branch absent. `UpdateRemoteBranchLeased` rejects an empty `expectSHA` as `ErrInvalidSHA`, so the fixer's plain non-forcing push is a sound substitute. This is the fixer's own change, not an error.
- A branch absent on origin can also mean origin deleted it after the task landed or was discarded (cleanup deletes task branches, `cleanupremotewarp.go`). Any hand-over where the last transition names a tip then re-creates a deleted task branch on origin.
- The give-up path removes it again, but only when the transition changes. A stable transition leaves a resurrected branch.
- This follows from the review's premise: `OriginCodeTip` reads absent as "tip missing". I can't show a path where origin's records still name a transition for a run whose branch was deliberately removed.

**F3 — no finding.**
- The second `recordLanding` after `archiveHeldParentMerge` reads `HeadSHA` from the same parent pair that was reset, so it yields the post-reset head.
- The landed-size range then becomes `parentHeadMerged..originTip`. That is coherent, since origin's tip holds the work.
- If the second `HeadSHA` read fails, the stale merge commit stays recorded, with a log warning only. This is minor and unchanged from the existing failure policy.

Calls made:
1. `sed` on `internal/pairteardown/handover.go` lines 80-140, `pr.go` lines 70-100, and a grep for `InsidePair`. I needed `HandOver`'s removal behaviour, the `PushBeam` caller's error handling and the other `PushRun` callers' `InsidePair` values, and my context did not show them.
