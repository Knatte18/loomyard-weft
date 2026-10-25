```yaml
lens: interactions
outcome: reported
```

Task: the interactions lens. I checked pairs of decisions that act on the same state.

Pairs checked:
- D4 continue pass (skips the reconcile) × D1 beam-owner probe × D13 long-lived wakeups: broken (F1)
- D4 pairteardown readers × D5 "empty status starts a task", with cancel left at `draft`: broken (F2)
- Goal "the orch stops resuming" × D4 "the orch resumes a halted run" × D6 × acceptance "halted gets no dobby pass": broken (F3)
- D4 "one check per Call" and Yield × D12 pause handling and the pass at lift: broken (F4)
- D1 runproc registration under the beam slug and its bound × D2 "a parent always has a dobby": broken (F5)
- D1 re-exec × D4 "first pass after a dobby starts runs the reconcile and Open-Window": broken (F6, ambiguous)
- D7 "one line per episode" × D4 and D5 "one notice per error or refusal text": broken (F7)
- D6 landing notice trimmed × D9 `landed` dropped: broken (F8, redundant)
- D5 "in flight, halted included" × D13 "backs off with no run in flight": broken (F9, ambiguous)
- D5 `max_runs` counting halted runs × D4 "no pass for halted runs": held (deliberate)
- D8 row notices wait for the stop chain × D9 kept Land and Teardown failures × D7: held
- D8 batten eligibility (landing halt handed back) × D8 Land failure held for the chain: held
- D9 api-error "next probe check" × D4 one check per pass with the memory record: held if the record covers it
- D9 driver-dead × D4 respawn cap: held
- D10 landing count from batten history keyed by run × D9 `landed` dropped × D4 passes: held
- D10 tidy agent's parent the nearest orch × D2 and D7 routing × Scope's orch rule: held
- D10 tidy agent's board edits × the board task guard and the run lock: held
- D12 "no run enters Land under a pause" × D4 Yield bound (InnerRun the only yielder): held
- D4 cancel marker × D5 cancel leaves `draft`: held
- D4 "a status write with no pass in flight triggers a pass" × D5 start and a hand resume holding the driver lock: held
- D5 conditions (owner this machine or none) × D1 dobby only on the beam owner's machine × D3: held
- D5 migration × running tasks holding a run status: held
- D2 `lyx adopt` ends with `lyx beam start` × D1 "a dobby owned elsewhere exits": held
- D4 `--window` retired × D6 stencil steps dropped × the beamcli and adopt callers: held
- D4 watched marker holding the dobby pid × D1 re-exec: held
- D8 copy of loom keys into `darn.yaml` × D8 loom.yaml keeps its meaning: held

Findings:

F1 — BLOCKING — design — D4 continue-pass paragraph × D1 × D13. D4 says that between full passes, "InnerRun's own owner check covers" ownership. That check is only due after a sleep the same process watched, a pause release, or a retry while owned elsewhere (`internal/battenshed/innerrun_owner.go:67-75`, `ownerSleep` and `ownerDue` in memory at `innerrun.go:165-170`). A short pass never watches a machine sleep that falls between passes. D13's dobby lives across the wake, so the next pass is a continue pass with no reconcile. D1's probe judges only the beam's owner, not the run's. So no reconcile runs for a run after a machine wake, against `PATTERN-owner-machine` ("acting on an entry after a wake ... origin reconcile first").

F2 — BLOCKING — design — D4 pairteardown paragraph × D5. D5 makes an empty status the start trigger and moves only `lyx batten cancel` to `draft`. D4 changes the claim settlement's liveness read but leaves its clear arm. `settleClaim` still clears the status of a run batten never seeded, or one that halted (paused, blocked, failed or awaiting) (`internal/pairteardown/claim.go:45-50`, written through `writeBoardClaim`, `claim.go:166-172`). So a `lyx fabric remove` of a halted run's pair leaves the task statusless, and its parent's dobby starts it afresh: the same undo D5's cancel rule exists to prevent.

F3 — MEDIUM — consistency — Goal × D4 × D6 × acceptance. The Goal says "the orch stops starting, resuming or cleaning up runs". D4 says "a halted or paused batten run waits for a resume by the orch or the operator". D4's continue-pass paragraph names "every pass that resumes a halted or paused batten run", which fits the acceptance criterion "a halted or paused batten run gets no pass from dobby" only if that pass is a hand pass, and the paragraph sits under dobby-scheduled passes without saying so. The record never states who resumes a halted run, nor that such a pass is never dobby's.

F4 — MEDIUM — design — D4 Yield and "one check per Call" × D12. D12 keeps "batten's existing pause handling holds every spawn". That handling is an in-process loop, `holdSpawnWhileCovered`, which sleeps per poll for the whole pause and while another machine owns the run (`innerrun_pause.go:51-73`). The driver's `PauseHold` also waits rather than returns (`sheddriver/drive.go` `Deps.PauseHold`). A pass that meets a covered spawn stays in flight for the pause, which contradicts D4's one check per Call. Because D4 never runs two passes of a slug at once, D12's "pass at once" on the lift becomes a no-op behind the blocked pass. The record does not say whether a covered spawn yields.

F5 — MEDIUM — design — D1 bound × D2. D1 bounds the end of a beam's dobby only by the cancel or owner pause of the beam's landing run. `runproc.End` of the beam slug is also reached from `lyx reed down` through `EndBeforeSessionDown` (`internal/pairteardown/skip.go:62-90`, ending at `teardown.go:569`). That path ends the beam's dobby while member runs still depend on it. D2's "a run's parent always has a running dobby" then fails until someone runs `lyx beam start`, and neither decision states this.

F6 — LOW — consistency — D1 re-exec × D4. D4 runs the reconcile and the Open-Window re-run "on the first pass after a dobby starts". D1 re-execs dobby on every binary change. The record does not say whether a re-exec counts as a start. If it does, every deploy re-runs Open-Window for every run with no client attached.

F7 — LOW — consistency — D7 × D4 × D5. D7 says dobby posts its own failures "one line per episode". D4 says "one notice per error text" and D5 says "one notice per refusal text". These are two dedup keys for the same notices, and "episode" is undefined for dobby's failures.

F8 — LOW — consistency — D6 × D9. D6 strips the `newly ready` part from the landing notice, which D9 drops entirely on success. D6's "landing notice" names a notice that no longer exists, so the two decisions act on one notice redundantly.

F9 — LOW — consistency — D5 × D13. D5 counts halted runs as "in flight" for `max_runs`. D13 backs dobby's timer off "with no run in flight". It is unclear whether a dobby holding only halted runs, which get no passes, backs off.

Tool calls: none.
