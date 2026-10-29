```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For every guard, refusal, branch and test row the cards add, I checked whether a real entry point can reach it and whether the guarded site recurs on a path no card guards.

**Findings**

- **BLOCKING** — Cards 11, 13, 15 (D5's remaining halt, the unresolvable token): the token halt can never be reached in production.
  - When no token resolves, `authRT.authenticate` returns the bare `resolveToken` error (`internal/githubclient/transport.go:119-121`).
  - `http.Client` wraps that error in a `*url.Error` whose `Op` is the method, not `parse`. `githubclient.IsTransient` reads that as transient (`internal/githubclient/transient.go`, the `urlErr.Op != "parse"` return), and so does `shedtransient.Class`.
  - Card 11 lists the `transientFailure` branch before the `ErrTokenUnresolvable` branch. Card 13 says a transient poll error "logs one Warn per episode and keeps waiting". Card 15 also routes failures through `transientFailure`.
  - So a missing token loops as a transient error or waits forever, and the `gh auth login` halt never fires.
  - The planned rows ("an unresolvable token halts") would pass only because the fake returns a bare `ErrTokenUnresolvable`.
  - `internal/hubissues/hubissues.go:197` checks `errors.Is(callErr, ErrTokenUnresolvable)` before anything else.
  - Fix: each card checks `errors.Is(err, githubclient.ErrTokenUnresolvable)` before the transient check. Its test row returns that error wrapped in a `*url.Error`.

- **MEDIUM** — Card 16 (D12, "covers `lym batten cancel`"): a plain `lym batten cancel` (no `--remove`) never closes the pull request.
  - Card 16 puts the close in `Teardown.RemovePair` and says every teardown route goes through it.
  - `lym batten cancel` without `--remove` keeps the pair: `pairteardown.CancelRequest{Keep: !remove}` (`internal/battencli/cancel.go:294`). It reaches `RemovePair` only through `removePairAndBranch` under `if remove` (`internal/battencli/cancel.go:422-423`); the flag defaults to false (`internal/battencli/cancelcmd.go:129`).
  - The cancel phase in `internal/pairteardown/cancel.go` is a second teardown route that no card guards.
  - Several halts name a plain `lym batten cancel <slug>` as their way forward (card 13's closed halt and missing-commits halt, card 8's field-`on` halt). After any of them the pull request stays open with no Warn.
  - Fix: card 16 also closes the pull request in the cancel phase, or states that a cancel keeping the pair leaves it open, with a test row for each.

- **MEDIUM** — Card 13 (pause yield) together with card 11's entry steps: a covering pause first runs the entry's push and GitHub calls.
  - On `Yield` the shed loop ends with `RunYielded` and the run exactly as it was (`internal/shedengine/run.go:494`, `:526-527`), so the next call re-enters `PRReview.Call` from the top.
  - Card 11's steps 1-8 run before the wait: they push the task branch with `PushTask`, call `FindOpen`/`Status`/`Edit` and write the record. The `PauseCovered` check only runs at the first wake inside `wait`.
  - So a call made while the pause still covers the run pushes and calls GitHub before it yields (or is held by the network gate). The guarded "nothing acts under a pause" state is not guarded at the row's entry.
  - Fix: card 13 reads `PauseCovered` in `Call` before step 5's push, with a test row in which a covered entry pushes nothing and calls no GitHub.

- **LOW** — Card 15 (`mergeOnGitHub` placement): the card never says where the GitHub merge sits relative to the merge-in-required retry.
  - It says `mergeOnGitHub` "replaces the local parent-side merge's effect" but not whether it runs after `recheckLock` at step 5/6, or how the retry path behaves (`internal/landingshed/finalize.go`, the `ErrMergeInRequired` branch re-runs `mergeAndGate`, `recheckLock` and `parentHandle.Merge`).
  - On a landing marked to merge on GitHub, the retry branch can reach the local `parentHandle.Merge` with no GitHub merge first, if the first attempt failed before `mergeOnGitHub`.
  - No row in `TestFinalize_PullRequest` reaches the retry under a GitHub landing.

- **LOW** — Card 13 (the closed-with-`ClosedForContinue` verdict versus the continue step): a re-entry can call `Close` on a pull request that is already closed.
  - On re-entry after a Finalize reroute back through PR-Review, the record holds both `Continue` and `ClosedForContinue`.
  - The card does not say whether the continue step (which calls `Close`) or the `Status` poll (closed plus `ClosedForContinue` is Done) runs first on a wake.
  - Closing the already-closed pull request adds a second comment or a GitHub error. The card should skip the continue step when `ClosedForContinue` is already set.

Tool calls made (my context lacked each of these):
1. grep of `Yield` in `shedengine` and of the token error path in `transport.go`/`hubissues.go`: to see what a yield does to the loop and how an unresolvable token surfaces.
2. Header of `pairteardown/cancel.go`, `battencli` cancel's `RemovePair` sites and `verifygate.go`'s seams: to check whether cancel goes through `RemovePair`.
3. `transport.go:112-128` and `battencli/cancel.go:585-606`: the token error return and the removal branch.
4. grep of `removePairAndBranch` and the `--remove` flag in `battencli`: to see that a plain cancel keeps the pair.
