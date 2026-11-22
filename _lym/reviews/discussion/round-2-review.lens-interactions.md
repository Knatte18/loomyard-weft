```yaml
lens: interactions
outcome: reported
```

Task: interactions lens. I checked pairs of decisions that act on the same state, then checked the acceptance criteria and constraints against each decision they touch.

**Pairs checked**
- D3 pause start condition × D7 reconcile pause: **broken** (F-a).
- D3/D7 pause gate × D1 "every commit reaches origin, none left out": **broken** (F-b).
- D5 dobby restart × D6 `CodeTipMissing` unset on the owner machine: **broken** (F-c).
- D5 refused-push gate (records branch only) × D3 pair pusher that also pushes the code branch: **broken** (F-d).
- D7 owned-elsewhere stop × D3 start conditions and D5 restarts: **broken** (F-e).
- D5 Worktree-Teardown push × D8 push-owed input unset: **broken, minor** (F-f).
- D5 no-owed-record mode × D8 act rule: held. Only `FailedPushesOwedBy` feeds `PushOwed` (runread.go:231-240), and it reads only owed classes (failedpush.go:276-292).
- D6 prime hand-over with `InsidePair` false × D3 pair pusher: held. The pair pusher exits when its worktree is gone. While it sleeps it holds no teardown lock (D5), so `HandOverTips`' exclusive take (handover.go:71) is not starved.
- D6 hand-over mark × D6 replay that leaves handed-over runs out: held as worded. Today `takeoverStop` returns `ErrHandedOver` for a marked run as a stop (takeover.go:124-126, pushclassified.go:165). D6 names the replay as a change site and says marked runs' commits are left out, so the next pass is not stopped.
- D6 × D3 runproc `HubOwner` key: held. `HandOver` ends the processes under the slug (handover.go:103), and the pusher is not under it. `EndSubject` spares `RolePush` (end.go:100).
- Weft-write-lock Constraint × D6 prime replay: held. The commit takes the lock and `retryWeftPush` try-acquires it (pushclassified.go:835-841).
- D1 `commit-records` verb runs no check × D7 check in the pusher: held.
- D5 Worktree-Teardown push × D3 pair pusher running concurrently: held. The push lock serializes them, and the tip re-check comes before removal.
- D5 Worktree-Teardown × D7 pause: held, since the teardown push bypasses the pusher's gate.
- D2 × D6 bound: held.
- Acceptance criterion D3 "a pause … none starts" × D7: broken through F-a.
- Acceptance criteria D5 "none for one that is level" and D6 "own machine acts in window" × D5 restart: broken through F-c.
- Acceptance criterion D7 "commit after pause not pushed until resume" × D1: broken through F-b; the cost is unstated.

**Findings**

**F-a — BLOCKING, design, `_lym/discussion/decision-record.md#D7`**
D7's pause and D3's gate name two different mechanisms, so the reconcile's pause does not stop the pusher.
- D7 says the run is paused "with the pause record `lym loom pause` writes", and that this is what the pre-push check does today.
- `lym loom pause` records a stop condition in the run's status, fired at a producer boundary (internal/loomcli/cli.go:430-436).
- The pre-push check actually pauses through `tendercli.PausePair` (prepush.go:77). That is `tenderengine.Pause`, which writes the hub pause record (tenderengine/pause.go:119-149).
- D3's start condition "no pause covers its run" reads that hub record's coverage (`pauseCovered`, landingdeps.go:692-705).

Built as D7 words it, the commit, the exit re-take and the dobby tick would all start a pusher after the reconcile's stop. That breaks acceptance criteria D7-2 and D3-1.

Fix: name the tender pause (`tendercli.PausePair`, the hub pause record) as the pause D7 writes and D3 reads. Drop the `lym loom pause` wording.

**F-b — MEDIUM, design, `_lym/discussion/decision-record.md#D3`**
D3's pause gate stops the pair pusher for every pause that covers the run, not only for D7's reconcile stop. This covers an operator's `lym tender pause` and a hub-scope pause, since `ScopeHub` covers every slug (tenderengine/pause.go:120-123).
- Every paused pair's commits, its last transition included, then stay off origin for the whole pause.
- Today `skipsCheck`'s `pauseCovers` branch lets a paused run's push through (prepush.go:99-101).
- The prime's pusher has no pause condition, so a paused batten run's prime records still push while its pair's do not.
- D7's bound states the hold only for a reconcile stop, so the clash with D1's "none left out" has no stated bound.

Fix: limit the gate to the pause the reconcile writes, or state and bound the hold for every pause, with the prime/pair asymmetry.

**F-c — BLOCKING, design, `_lym/discussion/decision-record.md#D5`**
D6 removes the code side's only existing restart, and D5 adds none in its place.
- D5's dobby restart reads only whether the local records branch is ahead.
- D6 leaves `CodeTipMissing` unset on the owner machine while its local code branch holds the tip.
- The existing code-tip push runs only for a `ReasonCodeTip` verdict (codetipsettle.go:117, 144-150), so it never fires there.

Counterexample: a pair pusher is killed after its records push lands and before its code push. The run is halted or stuck and gets no resume. The dobby tick reads the records branch as level and starts nothing. The code-tip settle does not fire. The code commits stay local until the next commit or resume. Meanwhile a read on another machine waits on `CodeTipMissing` until `CodeTipGrace` and then returns the run to its previous transition. This breaks D5's rationale that "every path that can leave a commit behind has a restart".

Fix: have the dobby tick count a pair as ahead when its local code branch is ahead of origin too, or keep the settle's push of a held tip on the owner machine.

**F-d — MEDIUM, consistency, `_lym/discussion/decision-record.md#D5`**
The refused-push gate checks only the records branch, but a pair pusher also pushes the code branch. Failed-push records are keyed by side and branch (pushclassified.go:457-470), and code-side refusals include `rewound`, `not_fast_forward` and `gone`. The record that a code-side refusal leaves therefore never matches the gate. Every later commit restarts the pusher, which pushes the records and meets the same refusal again. That is the restart loop round 1's interactions F1 was meant to close.

Fix: make the gate read the pair's code-branch refused record as well, or state the accepted behaviour.

**F-e — MEDIUM, design, `_lym/discussion/decision-record.md#D7`**
D7 says "an entry owned elsewhere is handed over, as the catch-up does today", but today the check only stops the push and leaves the hand-over to the dobby's catch-up (prepush.go:156-158; comment at prepush.go:116). If the pusher only stops, no hand-over mark and no pause is written. None of D3's start conditions then holds the pusher back: each commit, exit re-take and dobby tick starts a pusher that fetches, reconciles and stops again until the catch-up runs.

Fix: name who hands the run over inside the pusher, for example `pairteardown.HandOverOwnedElsewhere`, so that the hand-over mark gates restarts. Or add owned-elsewhere to D3's start conditions.

**F-f — LOW, consistency, `_lym/discussion/decision-record.md#D5`**
Worktree-Teardown's push runs "through `pairteardown.PushRun`" in the owed backoff, but D5 gives the no-record mode only to the pusher's push path. In the normal mode, `teardownLockOwedError`, `recordOwedRecordsPush` and the owed-class failures `pushClassified` records write owed-class records with `Runs` naming the slug (pushanchored.go:306-314, 378-380; pushclassified.go:472-474). While the teardown retries, those set `PushOwed` (runread.go:231-240), against D8's claim that the push-owed input stays unset.

Fix: state which mode the teardown push uses.

Tool calls: none; my context already held everything this lens needed.
