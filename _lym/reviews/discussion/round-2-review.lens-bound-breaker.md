```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guard, guarantee and claim about existing code in the decision record.

**Claims attacked**
- D1, no admission and no push in the transition: **held**. The pusher's own `PushPairAnchored` still takes `pauserecord.Admit` (pushanchored.go:161), but that is not the transition.
- D3, one pusher per worktree through a lock: **held** in the record's own terms.
- D3, the pusher registered under `runproc.HubOwner` survives the paths that end a run's processes: **held**. `EndSubject` spares `RolePush` (end.go:100). The run pass never reaches HubOwner. Tender's orphan reap exempts HubOwner (tenderengine/processes.go:119). Harden's reap ends owners of sandbox hubs only (hardenshed/reap.go:136–140).
- D3/D7, a pause covering the run stops the pusher on every start and continue path: **broken** (F-A, F-D).
- D5, "every path that can leave a commit behind has a restart": **broken** for the code side (F-B).
- D5, a refused push stops its worktree's pusher until the way forward clears it (plus the Auto-mode assumption): **broken** for a refused code-side push (F-C).
- D5, the pusher's push sets the upstream when it creates the branch: **held** (gitrepo/push.go:141, 227).
- D5, teardown's exclusive take never waits on the backoff: **held** for the owed backoff. The inner push backoffs (pushanchored.go:221–247, pushclassified.go:651–676, 287–310) still run under the shared teardown lock. They are bounded, and D6 treats a failed hand-over as owed, so a 30 s `HandOverTips` lock timeout (fabricengine/handover.go:71–77) heals on the next pass.
- D8, `runfsm`'s push-owed input stays unset while the pusher retries, and the criterion "with origin unreachable, the run walks its next rows and the dobby still schedules it": **broken** (F-E).
- D6, a pusher outside the pair survives the pair's removal: **held**.
- D6, a failed hand-over is retried "as a failed hand-over does today": **held**. A removal that fails after the mark is written meets `ErrHandedOver` (takeover.go:124–125) and is left to the dobby's leftover path, as today.
- D6, the prime's commits part cleanly by `Run-Transition` trailer: **held**. Both `commitRunRecords` and the seam commit through `battenCommitStatusDeps.Commit` with the trailer (commitstatus.go:166–169).
- Constraint, the weft write lock: **held** (commitweftpaths.go:41 and pushclassified.go:835–841).

**Findings**

1. **F-A** — BLOCKING, design, `decision-record.md#D7`
   - Summary: D7's pause is a false premise, so D3's pause gate never sees it. D7 says the reconcile pauses the run "with the pause record `lym loom pause` writes, as the pre-push check does today". But `lym loom pause` records a stop condition in the run's own status. The pre-push check instead pauses through `tendercli.PausePair`, which writes the hub's pause record. D3's start gate ("no pause covers its run") and today's `pauseCovered` read that hub record's coverage.
   - Counterexample: a plan that follows D7 writes a `lym loom pause` stop condition. The next commit or dobby tick reads the run as uncovered, starts the pusher, and pushes the stopped run's commits.
   - Evidence: prepush.go:77 (`tendercli.PausePair`); tenderengine/pause.go:119–149 (`pauserecord.Update`); loomcli/cli.go:430–435 (`pause` "records a stop condition the running phase machine fires"); loomcli/landingdeps.go:692–705 and pairteardown/runread.go:135–142 (coverage read from `pauserecord.ReadCoverage`).

2. **F-B** — BLOCKING, design, `decision-record.md#D5` (the dobby-tick bullet, with D6's code-tip bullet)
   - Summary: no restart covers a pair's code branch once its records side has landed. D5's dobby tick starts a pusher only for a records branch ahead of origin. D6 leaves `CodeTipMissing` unset on the owner machine when its local code branch holds the tip. Together they switch off the only existing push of a held tip, `settleCodeTip`, which acts only on a `ReasonCodeTip` verdict.
   - Counterexample: the pusher is killed (crash, sleep, reboot) between its records push and its code push, and the run is halted or stuck so no commit follows. The tick reads the records branch as level and the settle never fires. The code tip stays local indefinitely. Another machine sees `CodeTipMissing` and, past `CodeTipGrace`, returns the run to its previous transition. This contradicts D1's "with none left out".
   - Evidence: battencli/dobbyreads.go:259; battencli/codetipsettle.go:117, 144–150; pairteardown/runread.go:241–250; decision-record.md:103–107, 151–152.

3. **F-C** — MEDIUM, consistency, `decision-record.md#D5` (refused-push bullet, start conditions in D3, Auto-mode assumption)
   - Summary: the restart gate reads only "a refused-push record for its records branch". Failed-push records are keyed by side and branch, so a refused code-side push (rewound, `not_fast_forward`, auth) leaves its record on the code branch, which the gate never reads.
   - Counterexample: every later commit starts the pair's pusher. It pushes the records, meets the same code refusal and exits again. That breaks the Auto-mode assumption that a refused push stops its worktree's pusher until the way forward clears it.
   - Evidence: pushclassified.go:457–470 (the record is keyed by `j.side`/`j.branch`) and 690–716 (the rewound record on the code side); decision-record.md:67–68, 98, 195.

4. **F-D** — MEDIUM, design, `decision-record.md#D7` (and D3's pause start condition)
   - Summary: the new pause gate stops the pusher for every pause that covers the run, not only the reconcile's stop. That includes a hub-scope tender pause, which covers every pair. D7's bound names only the reconcile's pause and an unreachable origin. Today the `pauseCovers` branch skips the check and lets the push through.
   - Counterexample: an operator pauses a run, or the whole hub, and then adopts the run on another machine. The paused transition and the last rows' work never reached origin, so adopt re-runs older state. Today they would have been pushed.
   - Evidence: tenderengine/pause.go:120–123 (`ScopeHub` or slug); prepush.go:99–100; decision-record.md:67–68, 158–170.

5. **F-E** — MEDIUM, design, `decision-record.md#D8`
   - Summary: D8's claim that `runfsm`'s push-owed input stays unset is broken by every other caller of `PushPairAnchored` in its normal mode. For a pair, `opts.RunSlug` is set from the teardown slug, so a transport, deferred or `teardown_lock` record names the run. `FailedPushesOwedBy` then returns it, and `ReadRun` sets `PushOwed`.
   - The other callers: `lym fabric push`; the detached `SpawnStatusPush` children from tender, anchor change, follow-up and cleanup; the code-tip settle's push and give-up; and Worktree-Teardown's own `PushRun`, for which D5 names no mode.
   - Counterexample: with origin unreachable, one detached `lym fabric push` of the pair writes a transport record naming the slug. The act rule then holds the run (`ReasonPushOwed`), against the criterion "the run walks its next rows and the dobby still schedules it". The record clears only when the pusher lands.
   - Evidence: pushanchored.go:176–177; pushclassified.go:472–473, 626–627; failedpush.go:276–291; runread.go:231–240; fabriccli/weft_verbs.go:301; spawn.go:147–168; codetipsettle.go:45, 67, 79; decision-record.md:110–115, 172–178, 229.

**Tool calls**
1. Grepped fabriccli for `PushPairAnchored` and `TeardownWait`, tender for the orphan rule, gitrepo for upstream handling, and read dobbyreads.go:245–268: to find whether other `PushPairAnchored` callers write slug-named records and where `settleCodeTip` is called.
2. Read tenderengine/processes.go (the `Orphan` field and records loop), gitrepo/push.go (plain push lines) and fabriccli/weft_verbs.go:285–310: to judge whether tender's reap ends HubOwner pushers and whether `lym fabric push` goes through `PushPairAnchored`.
3. Read tenderengine/processes.go (`orphanOwner`) and gitrepo/push.go (upstream lines): to confirm HubOwner is exempt from the reap and that a push sets a missing upstream.
