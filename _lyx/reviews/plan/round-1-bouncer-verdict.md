---
verdict: BLOCKING
rationale: "The fixer resolved all fourteen round-1 findings, but card 8 misses a step-reachable GitHub arm: Publish.checkApproval's FindPullRequest (internal/landingshed/publish.go, the post-approve path) still turns a 5xx or timeout into Stuck, so github-api never reaches the classifier on the approve-then-start flow."
---

# Plan-Review round 1 — Bouncer verdict

## Round-1 findings against the revised plan

The fixer's revised plan (14 cards) resolves every round-1 finding.
I checked each fix against the code.

- **F1**: card 8 moves Publish's push, query and create arms and Finalize's parent-push arm onto the error path.
  `fabricengine.Fabric.PushBranch` → `PushWarpRebaseFreeAt` → `gitrepo.PushRebaseFree` wraps the `*GitError` with `%w`, and `loomcli/landingdeps.go` passes `pushBranch` through unwrapped, so card 2's classifier sees the error on both push arms.
  Rejected pushes return the bare `ErrPushRejected` and stay `Stuck`.
  The re-entry claims hold: `FindPullRequest` runs before `Create`, and `TestFinalize_AlreadyUpToDateMergeProceedsAsLanded` exists.
  One arm is still missing (below).
- **F2**: card 7 drops the bootstrap push claim and the integration test, and states that the `PreStep` mark is inert.
- **F3**: card 2 defines global-option skipping for `-c`/`-C` and adds the needed test rows. `gitexec.Run(args, cwd)` matches the integration test's call shape, and `testmain_test.go` supplies the hermetic `TestMain`.
- **F4**: card 13 adds `ErrPaneNotReady`, wrapped only by `requireReadyAgentPane`'s final not-ready return, and retries only on it. The named `run_test.go` tests exist.
- **F5–F9, F12–F14**: fixed as the fixer report describes. Every test the plan names exists in the code.
- **F10, F11**: the gitexec/githubclient split and the `driverResumeLine` card follow the granularity rule.

## Focus departure

The fixer departed from F1's written fix by keeping batten's Worktree-Create as a verdict.
**Ratified.**
`internal/battencli/wire.go`'s `createRefusal` doc says that "a rolled-back create keeps the branch it made".
So a re-step right after a transport failure there would stop on `ErrBranchExists`, which D3's single re-step would not clear.
Card 8 states the exclusion and the reason in `Publish.Call`'s doc comment.
The round-1 report had no `## Focus departures` of its own ("None").

## New findings

### [BLOCKING] Card 8 omits Publish's approval-path pull-request query

`Publish` makes two `FindPullRequest` calls, not one.
Step 3a's `checkApproval` (`internal/landingshed/publish.go`) queries the pull request whenever an approval record exists.
It turns a failure into `p.stuckOrCancelled(ctx, publishGitHubErrorReason("query existing pull request", err))`, and it runs before the merge-in, the push and the ordinary query.
This is the call the approve-then-start route reaches: `lyx loom approve`, then `lyx loom start` or batten's `Spawn`, which is the flow the decision record's batten acceptance criterion names.
Card 8 says "four arms" and lists only step 5's push, the step-7 query, the create, and Finalize's push.
A GitHub 503 or a `publishGitHubTimeout` deadline on the approval query would still persist `blocked`.
That breaks the Goal ("clears the stops that a single re-step would clear") and D2's `github-api` class on the one GitHub call every approved run makes.
Fix: card 8 also covers `checkApproval`'s query arm the same way, returning a `%w`-wrapped hard error when `shedtransient.Class` classifies it, and adds a fake-based test with an approval record and a 503 on the list call.
The ImpactSummary's arm count follows.

### [LOW] The new error branch must keep cancellation first

Card 8 puts the classified-error branch "after the arm's existing cancellation check".
The push, query and create arms have no check of their own: cancellation is checked inside `stuckOrCancelled`, which the new branch bypasses.
Card 3's `Canceled` rationale relies on "every landingshed arm checks its own context and returns the cancellation before any GitHub error".
A cancelled step whose GitHub call failed with a deadline would otherwise return a transient-classified error instead of the cancellation.
Fix: state that the new branch calls `cancelErr(ctx, <row>)` first and returns the cancellation when it is set, in every arm the card edits.
