# Plan round 2 — fixer report

Every finding's premise was checked against the code and the decision record, and all nine hold.
Eight are fixed as proposed and one (F2) is fixed differently.
`lyx loom validate-plan` passes after the edits (`ok: true`, informational `create-new-unit` findings only).

## Changes

- **F1** (MEDIUM, fixed as proposed): premise confirmed at `internal/shedengine/run.go:215-223`.
  The step-3 pause check persists `StatePaused` with `consumePause` and returns before any producer runs.
  The blocked branch at `run.go:330-335` passes `consumePause` false, and `goto.go:95` clears the flag.
  In card 9, `TestNew_PauseDuringLandReentersTheRowThroughGotoThenRun` now seeds batten at Land with no pause requested, so Land runs.
  The fake `Land.LandLock.Acquire` sets `pause_requested` on batten's status during Land's wait, and `Land.PauseRequested` reads that status.
  The goto-then-run sequence is unchanged.
- **F2** (MEDIUM, fixed differently): premise confirmed at `internal/gitrepo/push.go:78` and `:127`, where `rejected` in `rebaseRetryTriggers` matches every `! [remote rejected]` line, and no planned test drives a primitive.
  Card 1 now:
  - adds an unexported `Repo.pushSleep` field, which the primitives pass to `PushWithRetry`;
  - lists `internal/gitrepo#Repo` under Edit;
  - creates `internal/gitrepo/pushretry_integration_test.go`, an `integration`-tagged file in package `gitrepo`, holding one table test over `Push` and `PushRebaseFree`.

  The test asserts:
  - the error satisfies `ErrPushRetriesExhausted` and not `ErrPushRejected`;
  - the push ran `pushMaxAttempts` times, counted by the hook log, with the doubling sleeps;
  - no `pull --rebase` ran: HEAD is unchanged and no rebase is in progress.

  Not made: the proposed fixture, a branch `x/y` pushed to a remote holding `x`.
  A probe against a local bare remote with this machine's git gave `! [remote rejected] HEAD -> x/y (refname conflict)`.
  A held `refs/heads/z.lock` gave `(reference already exists)`.
  Card 1's classifier retries neither reason, so neither fixture can reach the exhausted path.
  Card 1 uses a bare remote with a `proc-receive` hook instead (`receive.procReceiveRefs=refs/heads`), which answered `! [remote rejected] HEAD -> w (cannot lock ref 'refs/heads/w': ...)` in the same probe.
  Also not made: rows in the existing `internal/gitrepo/push_test.go` table.
  That file is in package `gitrepo_test` and cannot set an unexported seam.
  The precedent for an `integration` test in the internal package is `internal/gitrepo/gogit_test.go`.
  The new test covers a behavior no existing test covers, the ordering at the primitives, so `PATTERN-test-economy` admits it.
  Card 1's budget sentence now counts three new top-level tests.
- **F3** (MEDIUM, fixed as proposed): `## The run loop` item 4 (stencil line 32) and `## Investigating a stop` (line 59) route every `[batten notice]` to a stop.
  Card 11 now has item 4 send a ready notice to `## After landing` and every other notice to Investigating a stop.
  Investigating a stop excludes the ready notice beside the parent-review request.
  The card also gains `wordingClaims` claims in both sections.
- **F4** (LOW, fixed as proposed): card 8 step 1 and step 2 now say a `PauseRequested` read error in either wait is a Stuck naming the step.
  The failing-step table now names the snapshot `ReadyTasks` error and the post-prune `ReadyTasks` error as separate rows, and its re-entry with the snapshot present sends the ready notice.
  It also gains a `PauseRequested` read error in each wait.
- **F5** (LOW, fixed as proposed): card 7's `TestWorktreeTeardown_FailureNoticeOncePerEpisode` gains a row where a `BattenHistoryLen` error after a queued notice still queues a notice, with a Warn.
- **F6** (LOW, fixed as proposed): card 11's `internal/orchcli/doc.go` paragraph now adds the hand deploy while `deploy` is empty to step 5.
  Step 4 now says a Publish verify failure reaches the orch only on its second failure in the batten run.
- **F7** (LOW, fixed as proposed): stencil line 136 reads "approve, land and clean up".
  Card 11 gains a bullet that drops "land and clean up" and names handling a Land or teardown failure notice instead, plus a `mustNot` claim in that section.
- **F8** (NIT, fixed as proposed): `contracts/stencils/claims_test.go:669` is the only `## After landing` claim on closing a run by hand, and line 668 is the kept drive-report claim.
  Card 11 now says "the `## After landing` claim".
- **F9** (NIT, fixed as proposed): `internal/shedrecipe/entries_batten.go`'s header says it "implements the four batten registry entries" and names them.
  Card 9 now places `landEntry` in that file, lists the file under Edit, and has its header stop counting and naming its entries.

## Disputed

None.

## Deferred

None.
