```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced the existing code paths that write or read the state D1 to D6 change.

**Paths traced, per decision**
- **D1** (transitions commit only)
  - batten seeded Worktree-Create commit (`arm.go`): survives.
  - `shedengine` persist calling `CommitStatus` on every write, skipped rows included: survives.
  - loom `refuseByAct` / `runOwnerVerdict` at start and resume (`internal/loomcli/ownercheck.go`, `ReconcileOrigin` with `JudgeOnly`): survives. These are pass-level checks, not transition checks, and D1 leaves them in place.
- **D2** (one pusher per run)
  - `PushPairAnchored`'s not-landed retry loop running under a driver that commits fast: survives, but there are two loops (see LOW 1).
  - Finalize's own `pushRecords` running at the same time as the pusher: survives. Both coalesce under the push lock.
  - Code-tip settle (`battencli/codetipsettle.go`, `ReadRun`'s `OriginCodeTip`): survives on the owner machine, which holds the tip and pushes it each tick. The window where records have landed but the code side has not grows (see LOW 2).
- **D3** (nothing stays unpushed)
  - dobby restart and resume restart: breaks when a failed-push record exists (see BLOCKING 1).
  - teardown archive push: its completeness promise breaks (see MEDIUM 1).
- **D4** (reconcile in the pusher)
  - the pause intent written by the pusher: survives, because tracked intents push themselves.
- **D5** (takeover)
  - `HandOver` calls `runproc.End` for the slug, and D2 registers the pusher in `runproc`. The record names the exclusion, so this survives.
  - `ReturnToPreviousTransition` on the adopting machine: unchanged.
- **D6** (owed push no longer holds the run)
  - batten overview and cancel aside: survive.
  - `dobbyreads.go`'s `DobbyRuns`, the `battencli/arm.go` pre-run judge and `loomcli` `refuseByAct`: these break (see BLOCKING 1).

**Findings**

1. BLOCKING — D6 and D3
   - **Summary:** the act rule's push-owed wait still stops the run through readers the record does not name.
   - **Evidence:** `runfsm.Act` answers `AnswerWait` / `ReasonPushOwed` first, ahead of every other case, whenever `in.PushOwed` is set.
     - `pairteardown/runread.go` fills that flag from `fabricengine.FailedPushesOwedBy`, which counts records of class transport, deferred and teardown-lock.
     - `DobbyRuns` judges with `runfsm.Aside{}` and `loomcli.hubRunVerdict` with `Aside{ShedPause: true}`; neither sets `PushOwed` aside.
     - `battencli/arm.go` also judges before a pass.
   - **Consequence:** if the pusher's owed push leaves a failed-push record, as the push path writes today, the dobby stops scheduling the run and a loom or batten resume is refused. That contradicts D6's "the run goes on".
   - **Second consequence:** D3's "a resume starts the pusher before the run acts" never runs in exactly the case it exists for.
   - **What the record lacks:** it says only that the push-owed input "stays for the batten overview and a cancel's aside". It must decide one of two things. Either the pusher's owed retries write no owed record that these readers see, or the dobby, pre-run and loom act readers set `PushOwed` aside. It must also say what adopt on another machine reads instead.
2. MEDIUM — D3, the teardown bullet
   - **Summary:** the archive push is not a completeness guarantee.
   - **Evidence:** `PATTERN-pair-remote` says a teardown's archive push of the weft branch "only logs a refusal", "since the removal archives and deletes the branch anyway".
   - **Consequence:** commits the pusher had not pushed can be dropped from origin with only a log line, against the operator's requirement that nothing is left out. The record has to choose: teardown either confirms the run's records reached origin, or treats a refused archive push like any other refused push, with a failed-push record and notice.
3. LOW — D2
   - **Summary:** there are two retry loops.
   - **Evidence:** `PushPairAnchored` already pushes again with `gitrepo.PushBackoffDelays()` while the records side holds unpushed commits after a push. D2 adds a re-read-and-push loop on top.
   - **Consequence:** under a burst of skipped-row commits, the inner backoff can spin before the outer loop sees the moved tip. The record should say which loop owns "the tip moved".
4. LOW — D2 and D5
   - **Summary:** the code-tip wait can fire more often.
   - **Evidence:** records land before code, so origin can name a `Warp-SHA` that origin does not hold yet. `ReadRun` then sets `CodeTipMissing`, and the dobby's `settleCodeTip` fetches on every tick of a `code_tip` verdict.
   - **Consequence:** the window existed with synchronous pushes too. With the run moving on while pushes trail, it is hit more often. On the owner machine it resolves by pushing the tip, so it survives, but each hit costs a dobby fetch per tick.

**Tool calls**
1. Grep for the code-tip settle and failed-push readers: needed to know who reads the async-pushed state; my context lacked them.
2. Read `battencli/codetipsettle.go` and `fabricengine/codetipsettle.go`: needed to see whether the code-tip wait survives records-first async pushes.
3. Grep and read `runfsm/act.go`'s use of `PushOwed` and `FailedPushesOwedBy`: needed to learn whether an owed push still makes the act rule wait.
4. Read the order of `runfsm.Act` and grep its callers: needed to learn which readers block a run on push-owed.
5. Grep `loomcli/ownercheck.go`, `battencli/dobbyreads.go` and `battencli/arm.go` for their asides: needed to confirm that those readers do not set `PushOwed` aside.
