---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/02-gitrepo-has-unpulled.md:6
    summary: Card 2 adds a runChecked-bound gitrepo method but does not target cmd/lyx/gitrepoboundary_test.go, whose pinned method set breaks go test and PATTERN-gitrepo-client-boundary
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-fabric-pair-anchored-push.md:74
    summary: No planned test reaches PushAnchored's new bounded lock wait or its retry, although Decisions 3 and 4 change its contract
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/03-fabric-pair-anchored-push.md:79
    summary: ImpactSummary of cards 3, 4 and 5 restates Intent and gives no caller count or scope bound
---

### [BLOCKING] Card 2 breaks the gitrepo client-boundary guard

**Location:** `_lyx/plan/02-gitrepo-has-unpulled.md:6` (the `**Edit:**` list)
**Issue:** Card 2 creates `Repo.HasUnpulled` on `runChecked` in `internal/gitrepo/push.go`.
`PATTERN-gitrepo-client-boundary` is a `(test)` entry, and `cmd/lyx/gitrepoboundary_test.go` (untagged) asserts set-equality between the methods of `Repo` whose bodies call `r.run` or `r.runChecked` and the literal `gitrepoPinnedRunBoundMethods` list.
`HasUnpulled` is not on that list, so `TestGitrepoBoundary_PinnedRunCallSites` fails under `go build ./... && go test ./...` at card 2's own commit, and stays red through the plan's `## verify:`.
No card targets that file.
The same entry makes a new CLI-bound call in `internal/gitrepo` a review obligation, and the Intent gives no reason `HasUnpulled` is not a go-git read, as the entry's "go-git owns local reads" half would otherwise require.
**Fix:** Add `cmd/lyx/gitrepoboundary_test.go` to card 2's `**Edit:**` list, with the Intent stating that `HasUnpulled` joins `gitrepoPinnedRunBoundMethods`.
State in the Intent that `HasUnpulled` is CLI-bound for the same reason as its mirror `HasUnpushed`, which `internal/gitrepo/doc.go` lists among the `runChecked` methods.

### [MEDIUM] PushAnchored's new lock and retry are not tested

**Location:** `_lyx/plan/03-fabric-pair-anchored-push.md:74`
**Issue:** Decision 3 puts `PushAnchored` under the absorbing lock with a bounded wait, and Decision 4 gives it the fetch-checked retry.
It is a separate exported entry, and batten's seam depends on it.
Card 3's tests cover the bounded wait, `SkipPush`/`SkipGit` and `LockWaitUnbounded` only through `PushPairAnchored`.
For `PushAnchored` the card only says it "gains its parameter in the existing tests", and `TestPushAnchored_DivergedWeftRemote_ReturnsErrPushRejectedUnwrapped` pins the unchanged bare sentinel after a real divergence.
Nothing reaches `PushAnchored` taking the lock, returning `ErrPushLockBusy` on expiry, or retrying and succeeding when the remote tip is contained in HEAD.
This is a coverage gap, not a redundancy: the shared unexported helpers are exercised, but the wiring of `PushAnchored` to them is not.
**Fix:** Extend the existing table-driven lock test with a `PushAnchored` row (a held lock and a bounded wait returns `ErrPushLockBusy` and the records remote does not move), and add a `PushAnchored` row to the retry step (a remote that declines once is pushed on the second attempt).
Add no new test function.

### [LOW] ImpactSummary restates Intent in cards 3, 4 and 5

**Location:** `_lyx/plan/03-fabric-pair-anchored-push.md:79`, `_lyx/plan/04-loom-pushes-both-sides.md:52`, `_lyx/plan/05-fabric-push-sync-both-sides.md:41`
**Issue:** The rubric asks for a one-line blast-radius conclusion, not a restatement of `Intent`.
Card 3's summary lists what the card adds and who contends on the lock, with no caller count.
Card 4's lists the changed surfaces.
Card 5's lists what the verbs now do, and does not say that `Fabric.PushWeft` loses its only production caller (the `push` verb) and keeps only a test caller.
Cards 1 and 2 state real conclusions ("additive ... no existing caller changes").
**Fix:** Rewrite the three summaries as blast-radius conclusions.
Card 3: `PushAnchored` gains a parameter, its three callers (two in `battencli`, one in `loomcli`) are all updated in the card, and `PushPairAnchored` has no caller until cards 4 and 5.
Card 4: one production caller moves to `PushPairAnchored`, one error message and one spec row split in two, the one test asserting the old text is updated, and `statuscommit` changes in comments only.
Card 5: `PushPairAnchored` gains its `push` caller, `spawnPush` has one caller, `Fabric.PushWeft` is left with a test caller only, and the bypass child is unchanged.

## Focus settlements

- **Fidelity.** Every Decision (1 to 6) and every Constraint is carried: Decisions 1, 3, 4 and the `PushAnchored` half of 3 by card 3, Decisions 2 and 6 by card 4, Decision 5 by card 5, the lock bound by card 1, and the retry's remote check by card 2.
  `StatusPushLockWait`, `LockWaitUnbounded` and `ErrPushLockBusy` are licensed: Decision 3 requires a bounded wait, Decisions 1 and 5 an unbounded one, and the lock-expiry error needs an identity for the card 3 and card 4 tests (card 4's lock-expiry row is the acceptance criterion for `commit-records`).
  The `PATTERN-push-both-sides` text equals Decision 6.
  Not a defect.
- **Granularity of card 3.** The two constants, the sentinel and `IsPushRejected` are the parameters and the error contract of `PushPairAnchored`, so they belong in its card.
  The battencli and loomcli call-site edits are forced by the `PushAnchored` signature change, which has to compile at card 3's commit.
  Not a defect.
- **Cards 3 and 4 on `loomcli/wiring.go`.** Card 4 uses card 3's handle, so the derived edge orders them and the edits cannot conflict.
  Card 3's pass of `StatusPushLockWait` is the compile-time step, and card 4 then replaces the call.
  Not a defect.
- **Test economy.** Cards 1 and 2 test new behavior no existing test covers (`TestTryAcquireWriteLock` and the push tests do not cover bounded acquisition or `HasUnpulled`).
  Card 3's `SkipGit`/`SkipPush` rows are new behavior of a new entry.
  Card 4's and card 5's divergence steps repeat card 3's assertions at the verb level, but the acceptance criteria require each verb to show them, and each owns its own envelope, so this is not a redundancy.
  The one real gap is F2.
- **Attack surface.** Card 1 states the bound (the caller's `wait`), card 2 states why the no-upstream fold is conservative, card 3 states the retry and lock bounds, card 4's disposition is unchanged with the lock bound in card 3, and card 5 states what the unconditional prime push lets through and what bounds it.
  Not a defect.
- **Verify coverage.** Every targeted package has its tests run: `internal/lock`, `internal/statuscommit` and the docs checks by the untagged `go test ./...`, the rest by the `-tags integration` line, and `internal/loomcli` also by the `tmux` line.
  No gap.
