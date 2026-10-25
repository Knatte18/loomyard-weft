```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: run each decision's mechanism end to end with the data each step actually gets, and check the end state against the decision's purpose.

**Decisions walked**
- D1, a dobby re-execs on a deploy while a pass is in flight. The pass runs in its own process group and the next pass reads the new pid from the state record. Purpose met.
- D2, `lyx orch start` on a hub with no prime owner. `beam start` records the owner, then starts the dobby; a second call finds it live. Purpose met.
- D3, a run whose task Land already pruned. It is found from the batten status file and its parent from the origin record. Purpose met.
- D4 yield, a child running at Plan-Write with no write. The continue pass's spawn check reads the confirmation marker as another pid's, finds the driver at work and adopts it. One check, `Yield`, exit 0. Purpose met.
- D4 continue pass after a laptop suspend with dobby still live. Purpose **not met** (F1).
- D4 memory record, a child whose owner idles holding the parent lock. Purpose **not met** (F2).
- D4/D12 spawn under a covering pause, an unconfirmed running child with a dead driver. Purpose **not met** (F3).
- D5, a release, then a start, then a halt, then `lyx fabric remove`. Purpose **not met** (F4).
- D6, a Land Done. The ready snapshot and routes are gone. Purpose met.
- D7, a refused start. Its notice goes up the beam chain to the orch. Purpose met.
- D8, a batten Land failure that the tackler escalates and the arbiter answers with `resume`. Purpose **not met** (F5).
- D9, a dead driver of a running child. Three respawns, then the driver-dead notice. Purpose met.
- D10, a restart after a landing. The count is keyed by run in the state record, so nothing is counted twice. Purpose met.
- D11, a comments-only change to `loom.yaml`. Silent. Purpose met.
- D12, a done child under a pause. It yields, and the lift triggers a pass that returns Done. Purpose met, apart from F3.
- D13, an idle dobby. The timer backs off to 60s and an event resets it. Purpose met.

**Findings**

F1 — BLOCKING — design — D4, continue-pass paragraph ("between those InnerRun's own owner check covers it").
A continue pass skips the reconcile, but InnerRun's owner check does not cover the gap between passes.
- The check becomes due only on three triggers: a sleep seen in-process (`ownerSleep.Observe` against `deps.Awake`), a pause release, or a retry while already owned elsewhere. Evidence: `internal/battenshed/innerrun_owner.go:67-75`.
- `Awake` is measured from process start (`internal/battenshed/innerrun.go`, in `NewInnerRun`, the `deps.Awake` default). A short pass process therefore never spans a sleep.
- After a suspend and wake with dobby still live, every pass is a continue pass, and no pass judges ownership against origin.
- This breaks `PATTERN-owner-machine`: acting on an entry after a wake needs the origin reconcile first.

F2 — MEDIUM — design — D4 memory-record paragraph and Open risks ("every field").
The record leaves the field list to be found out later, and the walk shows that choosing which state persists changes behaviour.
- The parent-lock idle bound lives in per-Call state, `childWait.lock` (`lockWatch.idle`), not in producer memory. Evidence: `internal/battenshed/innerrun_parentlock.go:14-19`, `internal/battenshed/innerrun.go`, `childWait` struct.
- With one check per Call, the idle watch restarts on every pass. A child's idle owner then never reaches the idle bound while passes come faster than the bound, so the lock is never released.
- Other per-Call fields behave the same way and are equally unnamed: `resumeRetryAt`, `notParkedLogged`, `busyLogged`, `lastProbe`, `sleep`.
- So do the producer fields: `noticeDelivery` attempts and `dropped`, `restepWatch` interval and attempts, `ownedBy`, `ownerNoticed`, `driverBusyNoticed`, `failures`.
- The decision does not say which of these persist. That is a deferred research question.

F3 — MEDIUM — design — D4 ("one check of its child per Call") and D12.
- The spawn path's `holdSpawnWhileCovered` loops, sleeping `pollInterval`, for as long as a pause covers the slug or `ownedBy` is set. Evidence: `internal/battenshed/innerrun_pause.go:51-70`.
- A pass that must spawn under a pause therefore stays in flight for the whole pause instead of yielding.
- D12's lift pass is then skipped as "a pass in flight".
- The record doesn't say this hold becomes a yield.

F4 — MEDIUM — design — D5 ("a task is started only from an empty status"; "a cancelled task is not started again").
- Pair teardown's claim settlement clears the board status of a removed pair whose batten run never seeded or halted. Evidence: `internal/pairteardown/claim.go:45-50` (`claimClear`), written through `writeBoardClaim` at `claim.go:166-172`.
- After `lyx fabric remove` of a halted run, the task therefore reads as empty status, and dobby starts a fresh run.
- D5 moves only cancel's own clear (`internal/battencli/cancelcmd.go:453-465`) to `draft`.

F5 — MEDIUM — design — D8 ("its spawn and its prompt move out of `internal/loomcli`").
An arbiter `resume` verdict needs its consume to be applied, and D8 doesn't say how batten gets one.
- The consume is `consumeArbiterAttempt`, which stays in `loomcli` and is built on `loomengine`'s attempt and envelope readers. Evidence: `internal/loomcli/arbiterconsume.go:1-24`.
- D8 names a move of the spawn and the prompt only.
- Batten's driver, which must not import `loomcli`, has no stated path to apply the verdict's goto. A batten resume verdict ends unapplied.

**Tool calls**
1. `internal/battencli/cancelcmd.go` `clearRunClaim` and cancel's claim wiring: to walk D5's cancel path. My context held only the `SetStatus` call site.
2. `internal/loomcli/arbiterconsume.go` header and api-error references in `battenshed` and `tenderengine`: to walk D8's resume consume and D9's api-error revive, which the manifest did not cover.
