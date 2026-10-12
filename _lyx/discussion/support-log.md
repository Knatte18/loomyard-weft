# Support log — lyx-halt

## Interview

Autonomous mode: no operator answered, so each turn below is the writer's own reading and pick.

1. Board read: the entry asks for a hub-wide `lyx pause` with an Esc after a short grace, recorded as ordered, and retires the boundary pause; it lists five open points and merges the `host-sleep-resume` seen case and issue #567. Related entries read: `creel`, `runs-overview`, `orch-split`, `run-message-routing`, `convergence-plan`, `spawn-audit-log`.
2. Code read, coarse: shuttle has `Runner.Interrupt` (the Esc choreography) and a stop verb that records `stopped`; batten's notices already measure idle in awake time through an `Awake` seam (commit fd201086d, 2026-10-10); the shed loop's step watchdog reads activity times off the wall clock; the only setter of `pause_requested` is the generic shed pause verb; goto's and the run lock's refusals name the bare pause as their way forward.
3. Shape: freeze, not stop. Since an Esc'd Claude turn just ends at the prompt and shuttle's Wait holds a turn end without outputs, a pause need not change any status: suppress the readers that would call it a failure, hold the entry points that would start network work, and type one line on resume. Picked over stopping runs (D5, D6).
4. Go producers: left running, since local work is safe across a suspend; network entry points are held instead (D4).
5. Orch: covered, via the watcher, with the single-writer pattern amended (D7).
6. Per-run form: kept, since the boundary pause it replaces was per run (D1).
7. CLI shape: `lyx resume` as an alias into the module, per `PATTERN-cli-cobra`'s alias clause (D1).
8. Boundary pause removal: bare form and `pause_requested` go; `--before`/`--after` stay, and `--after <current>` replaces the bare pause in the two ways forward (D8).
9. Unplanned sleep and #567: shared sleep detection, awake-time watchdog, one automatic continue across a sleep, burler's prompt check, and watchdog naming (D9, D10).
10. Lens review. Six forks were spawned, and all six returned. Their findings came back after the first gate pass and the delivery prompt, which said to change nothing, so they were folded in during parent-review round 1. One line per finding, duplicates merged:
    - premises, BLOCKING, D8: bare batten pause ends batten's wait mid-row, which `--after` cannot do — fixed: `lyx batten pause` and `pause_requested` stay (D8).
    - premises, MEDIUM, D10: the kill ends the step's process group, not the agent in its reed pane — fixed: the arming module stops the step's live shuttle runs after a kill, so the relaunch starts fresh (D10).
    - premises, MEDIUM, Constraints: stencils, specs, the recipe and the sandbox suites naming the bare pause were missing — fixed: listed under same-commit edits.
    - premises, LOW, Open risks: the watchdog premise holds — fixed: stated as fact in D9 and the risk removed.
    - premises, LOW, D3: `Runner.Interrupt` may refuse a pane that is not ready — fixed: reported as its own case.
    - goal-scope, MEDIUM, D6: resume is not handled per role, and forks are cut — fixed: the resume line names the cut tool call and forks; `interrupt_policy` is still not consulted, since no producer is re-invoked (D5).
    - goal-scope / pattern-fit / premises, MEDIUM, D8: batten and landingshed seams lose their trigger — fixed: the batten seam stays; landingshed's loom seam goes, with its bound stated (D8).
    - goal-scope, MEDIUM, D5: review rounds and crash-recovery diagnosis were not addressed — fixed: D5 states why they need no change of their own.
    - goal-scope, MEDIUM, criteria: no batten notice after an unplanned sleep was pinned — fixed: D9 bullet and a criterion.
    - goal-scope, MEDIUM, Scope: runs advance through local Go rows — fixed: stated in Scope Out.
    - goal-scope / scenarios, LOW-MEDIUM, D4: `SendMessage` bypass and an unneeded interrupt refusal — fixed: interrupt refusal dropped, `SendMessage` stated out of scope with its bound.
    - goal-scope / scenarios, MEDIUM, D9: shuttle deadline under sleep — fixed: deadline in awake time, and a criterion.
    - board-fit, MEDIUM: creel inherits the pause — fixed: Open risk naming creel.
    - board-fit, MEDIUM: `run-message-routing` relies on `lyx batten pause` as the urgent stop — fixed: batten pause stays (D8).
    - board-fit, MEDIUM: `runs-overview` cannot show a paused run — fixed: `paused_since` on status envelopes (D5) and an Open risk naming it.
    - board-fit, LOW: `orch-split`'s Go hub loop — fixed: a note for the plan writer.
    - board-fit, LOW: `verify-watchdog` / `driver-context` ownership — rejected: no live conflict, and the watchdog premise is now stated.
    - board-fit, LOW, process: ready marker read true at first await — rejected: the marker was written after the gate passed, as the procedure says.
    - pattern-fit, MEDIUM: `PATTERN-told-geometry`, the record path derived by engines — fixed: the path enters the geometries and is handed to readers (D2).
    - pattern-fit, MEDIUM: a leaf engine cannot run the sequence — fixed: split into the leaf `pauserecord` and the engine `pauseengine` (D2).
    - pattern-fit, MEDIUM: refusal-spec rows orphaned — fixed: Constraints names their rewrite, and D8 names the removed ones.
    - pattern-fit, LOW, D7: the verb vs. the orch pane — fixed: D3 says the verb never touches the orch.
    - pattern-fit, LOW: `lyx resume` may be agent-denied — fixed: Constraints asks the plan to check it.
    - pattern-fit, LOW, D10: owner of the stop call — fixed: loomcli's loop arming.
    - pattern-fit, LOW: `PATTERN-shed-verb-set` — fixed: D1 says it does not apply.
    - scenarios, BLOCKING: resume during the grace — fixed: per-session liveness check under the lock (D3).
    - scenarios, BLOCKING: overlapping entries — fixed: session ownership and transfer (D3, D6).
    - scenarios, BLOCKING: crash between Esc and record — fixed: intent recorded first, surplus line bounded (D3, D6).
    - scenarios, BLOCKING: spawn paths outside shuttle — fixed: D4 table.
    - scenarios, MEDIUM: parent lock held by a waiting network op — fixed: a per-run entry does not hold an op under a parent lock (D4).
    - scenarios, MEDIUM: a forgotten pause is silent — fixed: `lyx pause --status` and `paused_since`, plus an Open risk.
    - scenarios, MEDIUM: invalid input — fixed: D1 input rules and a criterion.
    - scenarios, MEDIUM: an Esc into a dialog, and a guarded paste refusal — fixed: D6 names a refused paste.
    - scenarios, MEDIUM: missing scenarios — fixed: criteria added.
    - scenarios, LOW: a held start's deadline — fixed: D4.
    - scenarios, LOW: 24-hour retention — fixed: 7 days, with its bound (D2).
    - scenarios, LOW: `pause_requested` at upgrade — rejected: the field stays (D8).
    - scenarios, LOW: who may call — fixed: D1 bounds an agent caller to its own pair's slug and refuses the rest.
    - leanness, MEDIUM: Constraints restated Decisions — fixed: only uncovered entries remain.
    - leanness, LOW: Goal restated Scope, a handled risk, D10's speculative clause, and an auto-mode bullet pointing at the support log — fixed: all cut.

## Rejected alternatives

- Stopping each run through shuttle's stop verb and relaunching on resume: kills the session's turn state and re-invokes producers, which is what `interrupt_policy` would have to arbitrate; the freeze needs neither.
- Setting each covered run's status to `paused`: Shed owns `state`, and a frozen producer is still running; a second writer would race the loop.
- A per-module pause flag (as webster had): many flags for one fact.
- A `pause` config module with a `grace_s` key: one value, better as a flag (YAGNI).
- `lyx pause --resume` or `lyx unpause`: less natural than `lyx resume`, which the alias clause allows.
- Killing or SIGSTOPping Go processes: a verify gate is rerunnable but a half-done landing is not; holding network entry points covers the real hazard.
- Excluding the orch: leaves the session that runs landings and pushes live across a suspend.
- A systemd-logind sleep inhibitor: Linux-only, a D-Bus dependency, and the operator chose an explicit verb.
- Removing `lyx batten pause` with the loom pause: it is batten's only mid-row stop, which the orch's hold-then-goto and `run-message-routing` rely on.
- Making `lyx pause <slug>` also set `pause_requested`, so it halts: the frozen producer never reaches a boundary, and the resume would then halt the run instead of continuing it.
- Refusing `lyx shuttle interrupt` under a pause: harmless during a pause, so the guard has no need.

## Review rounds

### Parent review round 1 (reject)

- F1, BLOCKING, D8 orphans the orch's hold-then-goto — fixed: `lyx batten pause` stays (mid-row, ends batten's wait), and the orch stencil's loom pause after Plan-Write becomes `lyx loom pause --after Plan-Write`. The orch stencil, the shed driver stencil and `contracts/stencils/claims_test.go` are listed as same-commit edits. `.lyx/orch/role.md` is rendered from the stencil, so it is not edited by hand.
- F2, BLOCKING, D4 holds only shuttle starts — fixed: the D4 table covers shuttle run starts (Burler relaunch included), `lyx reed resume`, batten's driver revive, `lyx loom start`, `lyx batten run`, `lyx loom resume`, `lyx shuttle send` and `lyx orch start`, each with its rule. Reed's server revive relaunches no strand. Criteria are added.
- F3, BLOCKING, lens findings not folded in — fixed: D3 and D6 settle a resume during the grace, overlapping entries and a pause killed mid-sequence. The medium findings are fixed or rejected in the Interview's turn 10.

## Question ledger

- Q1 Resume: is there a matching verb, and how does each role take up its turn? — Auto-pick: `lyx resume`, typing one fixed line into each session the pause interrupted (D6). Resolved.
- Q2 Is the orch paused too? — Auto-pick: yes, through the orch watcher (D7). Resolved.
- Q3 Does `lyx pause <slug>` pause one run? — Auto-pick: yes (D1). Resolved.
- Q4 Relation to shuttle's stop verb? — Auto-pick: the pause uses interrupt, not stop; the stop verb serves D10's watchdog record only. Resolved.
- Q5 Unplanned sleep detection, auto-resume, slept time out of idle timers? — Auto-pick: awake-versus-wall detection shared, no logind hook, one continue across a sleep, awake-time watchdog (D9). Resolved.
- Q6 Grace length? — Auto-pick: `--grace`, default 10 s, cap 60 s (D3). Resolved.
- Q7 Do Go producers stop? — Auto-pick: no; network entry points and new agent starts are held (D4). Resolved.
- Q8 Where does `lyx resume` live under `PATTERN-cli-cobra`? — Auto-pick: alias command into the pause module (D1). Resolved.
- Q9 What replaces the bare pause in goto's and the run lock's ways forward? — Auto-pick: `pause --after <current_producer>` for a loom run, `lyx batten pause` for a batten run (D8). Resolved.
- Q10 Does `lyx batten pause` go too? — Auto-pick: no, it stops a Go wait mid-row and nothing else does (D8). Resolved.
- Q11 How do hub and slug entries share a session? — Auto-pick: the first entry to interrupt a session owns it, and ownership moves on resume while another entry covers it (D3, D6). Resolved.
- Q12 Which agent spawn paths wait? — Auto-pick: the D4 table. Resolved.
- Whether the 2026-10-12 eleven-hour batten notice came from an older binary or a gap in the awake subtraction — the plan checks which and fixes the cause (D9). Resolved.
