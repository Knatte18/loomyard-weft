```yaml
lens: guard-reach
outcome: reported
```

Lens: guard-reach. For each guard, refusal, branch or test row a card adds, I checked whether a real entry point can reach the state it guards, and whether the guarded site recurs on a path the card leaves unguarded.

**Per guard, reachable or not**

- **Card 1**
  - Refused shapes (gitlab `ssh://` and `https://`, `ssh://git@github.com/loomyard`): reachable through test rows.
- **Card 2**
  - `LoadConfig` refusal of a value outside `on`/`off`/`required`: reachable through a hand-edited or `--set` value.
  - Local-layer refusal: reachable through configengine.
- **Card 3**
  - `normalizeTask` refusal: reachable through upsert.
  - `scopeChanged` run-lock refusal: reachable through an upsert on a held entry.
- **Card 5**
  - `StartSession` without `Unawaited`: fail-loud refusal with a test row.
  - `validate` refusal of `Unawaited` plus `OutputFiles`: reachable through a test row.
  - `stopOutcome` with an empty list: reachable through `StopStrand` on an unawaited run.
  - The vacuous-empty site does not recur on an unawaited record in any other reader: `sessionread.go:124` guards with `len&gt;0`, and the wait and attach sites see only validated non-empty specs.
- **Card 6**
  - `publish` deny rows: reachable.
  - The bound leaves out lym-internal push routes (finding 6).
- **Card 7**
  - `UpdateRecord` returning `ErrNoRecord`: reachable.
  - Refusal kinds: reachable through httptest.
  - `Open` with a non-GitHub origin: reachable.
- **Card 8**
  - Not-wanted, no-upstream and push-skip branches: reachable.
  - The "origin on github.com that `ParseOwnerRepo` refuses" halt: reach is undefined for every shape but `https://github.com/onlyowner` (finding 3).
- **Card 9**
  - Absent-record refusal: reachable by hand and by test. In production the session only exists after the record is written, so the agent itself rarely meets it. This is not a defect.
  - "Already set succeeds": reachable.
  - Decode-error envelope: reachable.
- **Card 11**
  - Nil-seam guard: a fail-loud wiring guard, exempt.
  - Unreadable `beam.yaml`, a resolver `Halt` from a field `on` with a local origin, the no-path close of an open recorded pull request (after the parent's mode is changed to `off`), push-failure halt, the `ErrTokenUnresolvable`-before-transient ordering (`http.Client` wraps a RoundTrip error in `*url.Error`, which `transient.go:41-44` reads as transient), the 404 halt and the transient error: all reachable.
  - `ClosedForContinue` re-entry: reachable through Finalize's `on_reroute` edge back through Describe.
- **Card 13**
  - Dirty tree waits, gate discard on a moved head or dirtied tree, mail once per head, "log when no strand" (always taken at card 13, before card 14 exists): reachable.
  - Merged verdicts and the approved verdict: reachable.
  - `holdWhileCovered` at entry: reachable only in the window after the driver's own `WaitUncovered` (`internal/loomcli/drivewiring.go:238`). Reachable, but narrow.
  - **Open with `Continue`: not reachable in production** (finding 1).
  - Closed with `Continue`: reachable, but it skips the clean and gated checks the open branch applies (finding 2).
- **Card 14**
  - Live reuse, resume by session id, `ErrNotStarted` versus other start failures, `SessionLive` error, stop on Done/Stuck/error, the pause stop and resume: reachable.
  - Nothing in the shellguard rows denies `lym loom pr-continue`, and `AudienceRole` is only an annotation, so the agent can run the verb.
- **Card 15**
  - No-record halt: reachable through `lym loom goto Finalize` or a lost record.
  - Closed-not-for-continue halt, the refusal kinds, the pending retry, the fast-forward failure and the nil `ParentMode` skip: reachable.
  - Merge-in-required retry after the GitHub merge: reachable through `merge.go:524`, which fires on a conflict.
- **Card 16**
  - Gone worktree, no record, merged or closed left alone, the failure warnings: reachable.
  - The guard test does not cover the issues-endpoint close (finding 4).
- **Card 17**
  - `RunningAtPRReviewIsAWait` and `BlockedAtPRReviewIsNoWait`: reachable.

**Findings**

1. **BLOCKING, design.** Card 13 (`PRReview.wait`, step 4 and the "open, with `Continue`" verdict): the operator's continue flag is never seen during a live wait.
   - `wait` takes `rec pullrequest.Record` by value at its start, and card 13 Uses `UpdateRecord` but never `ReadRecord`.
   - The card never says the wait re-reads the record at each wake.
   - In production, `lym loom pr-continue` (card 9) can only run once the session has started. Card 14 starts it after the record is written and before `wait`, so `Continue` always arrives mid-wait and the in-memory copy never holds it.
   - The test row seeds `Continue` before `Call`, so it passes while the production path is unreachable.
   - If the row persists `GatedHead`/`MailedHead` with a write of its stale copy rather than `UpdateRecord`'s read-mutate-write, it can also wipe a `Continue` the verb wrote.
   - Fix: re-read the record at each wake before the verdicts, persist only through `UpdateRecord`, and add a test row where the flag is written between two wakes.
   - Evidence: card 13's Create line for `wait` and its step 4; card 14's "`ensureSession`, called by `Call` after the record is written and before `wait`".
2. **MEDIUM, design.** Card 13 (closed verdict) and card 15 (`pullRequestLanding`, the `Continue` with a closed pull request branch): the continue precondition recurs unguarded.
   - The open-with-`Continue` verdict closes the pull request only on a clean tree whose head equals `GatedHead`.
   - The closed-with-`Continue` verdict ends Done on any close, with neither check. A close by a collaborator, or by the agent slipping past card 6's deny, therefore ends the review on a dirty tree or an ungated head.
   - Card 15 then lands that record locally. Finalize's merge guard and `finalize` gate bound the damage, but the card's own guard is bypassed.
   - Fix: apply the same clean-tree and gated-head check before Done, or accept this branch only when `ClosedForContinue` is already recorded.
   - Evidence: card 13's verdict "closed and not merged: Done when the record carries `ClosedForContinue` or `Continue`" against its "open, with `Continue`" verdict.
3. **MEDIUM, design.** Card 8 (`ResolvePullRequest`, origin checks) with card 1 (`ownerRepoPath`): the D4 halt for "an origin on github.com that `ParseOwnerRepo` refuses" has undefined reach.
   - The card never says how the resolver tells "host is not github.com" apart from "github.com, but refused".
   - Card 1 keeps the one "not a github.com remote URL" error for gitlab URLs and for refused github.com shapes alike, such as `https://@github.com/o/r`, `ssh://git@github.com:22/o/r.git` and `git://github.com/o/r`.
   - Only `https://github.com/onlyowner` is pinned. The rest may be read as a non-GitHub host and become a silent local "no" under parent `on`, which D4 forbids.
   - Fix: name the host test, for example the URL's host after any scheme and user-info, or the `git@host:` form, and add test rows for those shapes.
   - Evidence: `internal/githubclient/parseownerrepo.go:46-47`; card 8's Intent bullets on origin facts; card 1's Intent on `https://@github.com/o/r`.
4. **LOW, design.** Card 16 (`TestPullRequestGuard_OnlyLandingAndTeardown`): the guard scans only `.PullRequests.Create(`, `.PullRequests.Edit(` and `.PullRequests.Merge(`.
   - A pull request is also an issue, so `.Issues.Edit(` with state closed closes it outside the three permitted sites with nothing to flag it.
   - The import guard covers only importers of `internal/pullrequest`, not a direct go-github caller.
   - Evidence: card 16's Intent paragraph on the guard test.
5. **LOW, design.** Card 6 (`rowsAfterRawGo` publish row comment, the bound): it lists only shell forms that slip the deny (`bash -c`, a script, `env`, an alias, `hub`).
   - It leaves out the lym verbs that push the task branch from the session unguarded: `lym fabric push`, and the detached both-sides push `lym loom commit-records` fires.
   - D8's bound names the detached push, but the card's stated bound under-reports what the guard lets through.
   - Evidence: card 6's Intent, the row comment sentence; decision record D8's Bound.

Tool calls:
- `grep` over shellguard rows and `clihelp.AudienceRole`: to check whether any deny or audience gate blocks the agent's `lym loom pr-continue`; the manifest held neither.
- `grep` of `OutputFiles` readers in shuttleengine and the `ErrMergeInRequired` definition: to see whether the vacuous-empty site recurs on an unawaited record, and what reaches the merge-in-required retry.
- `grep` of the `ErrMergeInRequired` construction site and the pause-coverage reads in the driver and loomcli: to trace whether `holdWhileCovered` at `Call` entry can see a covered state past the driver's own wait.
