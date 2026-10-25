```yaml
lens: interactions
outcome: reported
```

The interactions lens found 2 BLOCKING, 6 MEDIUM and 2 LOW findings.

**BLOCKING — D4 × D7/D9 (passes for "every run in flight")**
- **Summary:** D4 has dobby run a pass on every probe tick, on every status write and at startup for each run it drives. A pass is `lyx batten run <slug>`, and that verb resumes a blocked or paused batten run. So every halt is resumed again on the next probe, and nothing in the record stops it.
- **Evidence:**
  - `internal/battencli/cli.go`: `run` starts or resumes a run that is "blocked, or paused", "the everyday resume path".
  - The Worktree-Create precondition halt and hand-back halts (which D7 says go to the orch) and Land's pause-blocked halt (whose way back is `goto --to Land`) would all be re-stepped every `notice_probe_s`.
  - Each re-entry appends batten history. The row-notice episode is keyed by history length (battenshed doc), so D9's "kept" Land and Worktree-Teardown failure notices would repeat on every probe.
- **Fix:** D4 must say that dobby runs passes only while batten's own state is `running`. A halted batten run then waits for the orch's or operator's resume.

**BLOCKING — D5 × the existing status-clearing paths (cancel, hand `set-status`) under D4's start rule**
- **Summary:** D5 starts any task whose status is empty. `lyx batten cancel <slug>` clears the status (boardengine doc: "ended with `lyx batten cancel`…, which clears the status"), and so does a hand `set-status` on a live run. Dobby would restart a cancelled task on its next tick.
- **Evidence:** In D5, "empty status" is the only gate between a cancelled task and a fresh start, and the record never says what cancel leaves behind. A related problem: "status-only change revises nothing" (boardengine doc), so the operator cannot re-arm a refused start by touching the status.

**MEDIUM — D1 × D3 (dobby registered under the beam slug)**
- **Summary:** The beam's own landing run (`lyx beam land`) is a batten run under the same slug. So `runproc.End(<beam slug>)` from that run's cancel, an owner pause, or a failed-land cleanup also ends the beam's dobby while the beam still exists. D1 claims only the pair teardown reaches it.
- **Evidence:**
  - D1: "nothing that ends a member run's processes reaches it". That holds for members, but not for the beam's own run.
  - The runproc doc: `End` ends every live record of an owner except the caller and its ancestors.

**MEDIUM — D5 refused-start latch × the ways forward for a refusal**
- **Summary:** After a refusal, the latch waits for the entry's `revised` time to change. Several ways forward never move it, so the task stays latched for good, and the latch's own persistence is unstated.
- **Evidence:**
  - `lyx adopt` and an owner write move no `revised` time, and status changes revise nothing.
  - A cause that passes by itself, such as origin being unreachable at a fresh start (batten "pulls the board instead, which needs origin"), also leaves the entry unrevised.
  - D7 puts the episode marker under `.lyx/beam/`, but D5 doesn't say whether the latch survives a dobby restart.

**MEDIUM — D7 × D8 × D9 (Land and Worktree-Teardown failure notices)**
- **Summary:** D7 says a batten stop meets the tackler and arbiter "before anything reaches the orch", and D9 keeps the Land and Teardown failure notices "with today's routing". But today's row notifier posts on the row's failure itself, ahead of the driver's stop chain. So the orch is told while D8's arbiter may still be settling the stop.
- **Evidence:** battenshed doc, rownotice: "Worktree-Teardown on every failure it returns, and the Land row on the same terms". Only InnerRun's child notices have an arbiter hold, not these rows.

**MEDIUM — D3 × D4 (retiring `--window`)**
- **Summary:** D4 retires `--window`. But `lyx beam land` and `lyx adopt` start runs through `battencli.ResumeInWindow`, the `--window` body run in-process, and D3 says the parent's dobby drives the beam's landing run. The record doesn't say whether these verbs now hand the slug to dobby or still run a window, so two drivers of one run (verb plus dobby) or a dead path both stay possible.
- **Evidence:** `internal/battencli/cli.go` `ResumeInWindow` ("the in-process resume an adopting verb runs"); `beamcli` doc: land "starts the landing with `battencli.ResumeInWindow`".

**MEDIUM — D9 × D10 (landing count)**
- **Summary:** D9 drops the `landed` notice, and D10 counts landings in dobby's state. The record doesn't say how dobby observes a landing. If a dobby dies while the Land/Teardown pass is in flight, the restarted dobby no longer sees the run in flight and can miss the count.
- **Evidence:** In D4, a pass runs in its own process group and outlives dobby, while D10 counts only "landings of runs it drives".

**MEDIUM — D2 × D5 (two prime dobbies)**
- **Summary:** D2 starts a dobby for "every beam worktree this machine owns", and the prime counts as a beam. With no recorded prime owner, two machines' `lyx orch start` can both run prime dobbies, and both can start the same ownerless task. Batten's fresh start only refuses a task "another machine owns" after its board pull, which leaves a race. The record doesn't define prime ownership for D2.
- **Evidence:** battenshed doc: the prime owner is recorded on resume "when none is recorded". D5: "owner is this machine or none".

**LOW — D1 re-exec × D4 watched marker**
- **Summary:** The marker holds dobby's pid. A re-exec keeps the pid on Linux but not where re-exec spawns a new process, as on Windows, so the marker reads stale until the next pass rewrites it.
- **Evidence:** D1 "re-execs itself"; D4 "marker holds the pid of the dobby".

**LOW — D5 (`draft` default)**
- **Summary:** The default covers "an upsert that creates a task" and `promote`. `upsert-batch` and `merge`, which also create tasks, aren't named, so a batch-created task could start at once.
- **Evidence:** The board verbs list includes `upsert-batch` and `merge`.

**Pairs checked:**
- D1×D2 (start, lock, ownership): held.
- D1×D3 (registration slug vs the beam's own land run): broken.
- D1×D4 (re-exec vs watched pid): held on Linux; LOW.
- D1×D10 (tidy session vs beam teardown): held.
- D2×D5 (prime dobby per machine vs ownerless start): broken (MEDIUM).
- D2×`BeamLandRefusal` and the member start guard: held.
- D3×D4 (`ResumeInWindow` callers): broken (MEDIUM).
- D4×D5 (driver lock as in-flight guard, start pass): held.
- D4×D7/D9 (passes on halted batten runs): broken (BLOCKING).
- D4×D9 api-error (revive-sent fact in the memory record): held, if the record carries it.
- D4×D12 (passes under pause): held, except for the halted-Land case above.
- D4×D13 (pass per status write, coalesced to one a second): held; cost only.
- D5×D6 (readiness from the board): held.
- D5×cancel and hand `set-status` (empty status): broken (BLOCKING).
- D5 latch×`revised`: broken (MEDIUM).
- D7×D8 (stop chain): held for InnerRun; broken for row notices (MEDIUM).
- D8×loom/darn overrides and the sparse hub base: held.
- D9×D10 (landing count): broken (MEDIUM).
- D9×D11 (config notice carrier): held, since Finalize posts through its own Notify, not the landed notice.
- D12×D4 lifted-pause pass: held for running runs; a Land blocked by the pause needs `goto`.
- Acceptance criteria: "A pair teardown of a landed beam ends its dobby" holds; "never run twice at once" holds; "After a lifted pause, every covered run gets a pass" conflicts with the blocking finding unless restricted to running batten runs.
- Constraints: the `PATTERN-pair-teardown` cwd rule holds; `PATTERN-run-process-owner` holds; `PATTERN-shed-driver` holds.

**Tool calls:**
1. Grepped `internal/landingshed/finalize.go` for the config notice's carrier, and `internal/battencli/cli.go` for whether `run` resumes halted runs and for `ResumeInWindow`. The context held neither.
