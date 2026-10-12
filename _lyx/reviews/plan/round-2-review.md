---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/07-fabric-parent-reads.md (Fabric.FastForwardPair)
    summary: FastForwardPair is a mutating fabric verb whose planned result neither carries a Mutations record nor takes the SyncOptions its named arms need, against PATTERN-mutation-record
    fix: >-
      In card 7, re-sign the Create glyph to `plan:internal/fabricengine#Fabric.FastForwardPair` -> `func (f *Fabric) FastForwardPair(opts SyncOptions) (res FastForwardResult, err error)` and add a Create glyph `plan:internal/fabricengine#FastForwardResult` -> `type FastForwardResult struct`, which embeds `MutationRecord` and carries `Refusal FastForwardRefusal` and `NewHead string`. State in the Intent that the method accumulates a `*Mutations` record with `NewMutations`, passes it to `ResetHard(rec, upstreamSHA)`, records the advance as `Pull` does with `recordWarpAdvance`, passes `opts` to `PullWeft(opts)`, and returns the record in the result even on a refusal (where it is empty). The integration test asserts that a fast-forward's result holds the reset entry and that a refusal's result holds none. In card 12, `parentPuller` and `syncParent` read `FastForwardResult.Refusal`, and Merge-In passes the landing deps' sync options.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/22-status-lock-wait.md
    summary: Card 22 adds a fifth wait to the lyx loom status waiting key but leaves loom-status-spec.md, which names four waits, their order and their render forms, unedited
    fix: >-
      Add `contracts/specs/loom-status-spec.md` to card 22's Edit list. In its Intent, have the spec's `## The waiting key` section count five waits, list the parent lock wait with its order and its render form `<producer>: waiting for the parent lock on <branch> held by <slug> <elapsed>`, and state that a lock wait record whose writer is dead is ignored. Also state where `parentLockWaiting` sits relative to `gateWaiting`, which today wraps the whole chain and never hides the rest. Either it goes inside `gateWaiting`, ahead of the verify note, or the spec's gate-wait sentence changes to match.
---

### [BLOCKING] FastForwardPair breaks the mutation-record contract

**Location:** `_lyx/plan/07-fabric-parent-reads.md`, the `Fabric.FastForwardPair` glyph and its Intent bullet.

**Issue:** `PATTERN-mutation-record` requires every mutating fabric verb to accumulate a `*Mutations` record and every mutating result type to embed `MutationRecord`.
Every composite mutating `Fabric` method follows that contract: `Pull`, `Merge`, `MergeIn`, `Commit`, `PushWeft` and `PushBranch` all return a result that embeds `MutationRecord`, and the primitives (`ResetHard`, `ResetPairCode`) take the caller's `rec`.
Card 7 plans `FastForwardPair() (FastForwardRefusal, error)`, which moves the parent pair's warp through `ResetHard` and its weft through `PullWeft`.
That signature has no result that can carry a record, so the reset entry `ResetHard` appends is created and then dropped.
The same signature takes no `SyncOptions`, but `PullWeft(opts SyncOptions)` (`internal/fabricengine/recordsgit.go:338`) needs one for its `SkipGit` gating, so the implementer would have to invent the options inside fabric.
The rest of the fast-forward design is sound (see the focus verdict below); only the result and input shape are wrong.

### [MEDIUM] The status spec is not told about the lock wait

**Location:** `_lyx/plan/22-status-lock-wait.md`.

**Issue:** The decision record's scope puts "the loom status spec" in the same commit as the code.
`contracts/specs/loom-status-spec.md:79-100` defines the `waiting` key as "one of four waits", gives their read order and gives each one's render form.
Card 22 adds a fifth wait with its own render form, but only card 3 edits that spec, and only for the reroute outcome and history fields.
After card 22 the spec therefore describes four waits while the code renders five.
Card 22 also says it "puts it first in the chain".
But `reviewWaitingFor` (`internal/loomcli/reviewstatus.go:158-175`) makes `gateWaiting` the outermost wrapper, and the spec states that a gate wait never hides the rest.
Neither the card nor the spec says whether a lock wait now hides a gate wait.

## Focus verdicts

- **Lock wait counts as activity for the step idle watchdog: not a defect.**
  Card 22 edits `internal/loomcli#agentActivityFor`, which today reads only the verify marker and shuttle's markers and activity (`internal/loomcli/looparm.go:52-83`).
  The card reads the live wait record right after the verify marker and returns `time.Now(), true`.
  That record is written by the step process through `NewWait` (card 2) at `parentlock.WaitRecordPath(loomengine.LoomScratchDir(loc))` (card 11), so its writer is alive for the whole wait.
  The card also names the test that reaches the behaviour, `TestAgentActivity_LiveLockWaitReadsAsActivity`.
  `specFor` (`internal/loomcli/arm.go:321`) is the one place that wires this reading into the loop for both loom and darn.
- **FastForwardPair enforces the fast-forward inside itself: not a defect on that question.**
  Card 7 never calls `Pull` (`internal/fabricengine/pull.go:198`), whose non-fast-forward arm hard-resets or re-anchors.
  The order is fixed: it refuses on a fabric merge record (as `Pull` does at `pull.go:209`), fetches once and refuses on tracked dirt.
  It then refuses with `FastForwardNotFastForward` unless the local head is an ancestor of the freshly fetched upstream SHA, and only then calls `ResetHard` to that SHA.
  So the ancestry check and the reset read the same SHA, and an upstream force-pushed after Merge-In's `ReadParentTips` is refused.
  The shape defect in F1 is separate from this question.
- **The parent lock queue wait is in the daemon-wakeups guard: covered.**
  Card 2 puts `Store.Acquire` in `internal/parentlock/acquire.go` and adds that file to `cmd/lyx#unboundedWaitFiles` (`cmd/lyx/daemonwakeups_test.go:22`).
  The same card adds the wait, with its one-second-to-thirty backoff, to the What counts list in `pattern/PATTERN-daemon-wakeups.md`.
  Card 5's no-kill loop watch lives in `internal/shedverbs/loopwatchdog.go`, which is already on the list.
  Card 24's lock step lives in `internal/battenshed/innerrun.go`, which is also already listed.
- **Every prescribed refusal has a test that reaches it in its owning package: yes.**
  - The fabric merge-in-progress mapping is reached in `fabricengine` by card 7's integration row: a conflicted fabric merge left unresolved leaves the record, which `Fabric.Merge` writes through `saveMergeState` (`internal/fabricengine/merge.go:242`).
  - In `landingshed`, the same mapping is reached by card 12's `mergein_test.go` row for a merge in progress, through the fake `parentPuller`.
  - The dirty, not-fast-forward and no-parent-pair states have rows in both cards.
  - The guard's paused and unreadable reasons are covered by card 11, and the reroute cap and the missing-`on_reroute` wiring fault by card 3.
  - The lock verbs' refusals are covered by card 25, and the new seam values and keys by card 16.
  - The new nil-dep constructor refusals in cards 12 and 20 are wiring guards in `refusal-spec.md`'s sense (`contracts/specs/refusal-spec.md:31`), which take no per-row test.
    That covers Publish's, whose `TestNewPublish_Refusals` card 20 does not edit.
