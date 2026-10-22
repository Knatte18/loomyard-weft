# Support log — orch-strand-cycle

## Interview

Autonomous run: no operator answers; every pick below is an auto-pick recorded in the question ledger.
No advisors were seated.

### Turn 1 — exploration

Read the board entry (sections 1–4 and the two folded notes) and the entries it touches: `loop-exit-halt-names-its-cause` (owns the `loop-exited` halt of #811), `batten-cancel-verb` (depends on this task), `creel-agents` and `spawn-audit-log` (no overlap).
Read reed's status, listing, resume, launch and add paths; orch's status, start, refresh, distill and the watcher's idle and note phases; creel's delivery error mapping and strand listing; seatengine's name check; issue #746.

Findings that shaped the design:

- Reed's listings are already scoped to the reed session, so the placement condition is mostly met today; the gap is a removal or relaunch killing a server-global pane id that now sits elsewhere.
- The launch line is typed into an interactive shell and sourced, so an exited engine leaves a present, not-dead pane running the shell.
- `lyx orch start` already spawns a watcher for a live strand; `status` does not.
- The watcher's cycle start persists the phase and clears the request marker in two separate steps.
- `ErrSentTextInDraft` and `ErrForeignBoxText` wrap `ErrSessionBusy`, which creel maps onto `ErrBusy`, "nothing was typed", although both are read after the paste.
- Creel's `Strands` fails the whole call on one worktree that does not resolve.

### Turn 2 — design batch (auto-picked)

Q1 liveness test for "the engine is foreground": compare `#{pane_current_command}` against the pane's shell (recommended), against a spawner-supplied engine name, or check the shell's child processes. Picked the shell comparison.
Q2 boot window: launch grace on a recorded launch time (recommended), or two readings apart. Picked the grace, 30 seconds.
Q3 Windows: skip condition 3 (recommended) or apply it. Picked skip.
Q4 watcher restart at `status`: under a `start.lock` try-lock, only for a live strand (recommended). Picked.
Q5 note gate: turn end plus non-empty file, re-read from the phase's events offset (recommended). Picked.
Q6 withdraw race: serialize on the state lock with the watcher's cycle start (recommended), or accept the race. Picked the lock.
Q7 name reclaim site: reed's add (recommended), or each respawning caller. Picked reed's add, exact name only.
Q8 #828 fix: remap post-paste errors to `ErrNotLanded` and withdraw a queued `/compact` once the boundary is seen (recommended). Picked both.

### Turn 3 — lens review

All seven forks returned. Findings, merged across lenses:

- walkthrough BLOCKING, D6 never fires on a role-numbered add since numbering runs past the dead holder — fixed: reclaim the un-numbered role name before numbering.
- existing-paths BLOCKING, condition 3 in `aliveIDSet` would read Selvage dead — fixed: D1 names its readers; pane-level checks (Selvage, pane options, title repair) keep conditions 1–2.
- existing-paths MEDIUM, shuttle stop leaves a shell-only shared-window pane — fixed: D1 consequence, stop removes a present strand.
- interactions MEDIUM + bound-breaker LOW, a strand with no launch time kept #746 alive — fixed: no launch time gets condition 3 with no grace.
- interactions LOW + walkthrough LOW + bound-breaker MEDIUM, refused requested-cycle entry leaves the creel hold — fixed: D5 releases it.
- bound-breaker MEDIUM, locked check by presence lets a new refresh be consumed by the old request — fixed: compare mode and request time.
- walkthrough LOW + existing-paths LOW + bound-breaker LOW, withdraw answer undefined in a hard/soft cycle with no request — fixed: `withdrawn: false` with phase and trigger.
- bound-breaker MEDIUM, `status` restart races `stop` — fixed: stop takes `start.lock`.
- bound-breaker MEDIUM, pending-send bound false for `ErrForeignBoxText` and recordless sends — fixed: bound states such an inject is lost, not retyped.
- bound-breaker MEDIUM + goal-scope MEDIUM, worktree skip too narrow ("not a git repository" in watch.log) — fixed: skip covers a gone directory or a not-a-git-repository resolve.
- bound-breaker MEDIUM, creel queue of the reclaimed name — fixed: D6 states mail is name-addressed and goes to the new strand.
- bound-breaker LOW, reclaim during a pair pause — fixed: reclaim skips while the pair is paused.
- bound-breaker LOW + goal-scope LOW, #828 cause stated without evidence — fixed: journal and watch.log evidence cited.
- interactions LOW, receiver-wide creel withdraw drops a resume line — fixed: D7 clears the resume-noticed mark.
- goal-scope MEDIUM, window and "claude" departures not called departures; Windows skip — fixed: D1 lists the departures with reasons.
- goal-scope LOW, status restart narrowing — fixed: listed under auto-mode assumptions.
- goal-scope LOW, rules section removal is a soft deferral — fixed: the PR body names the orch step; the code change cannot write Bolt.
- board-fit MEDIUM, Resume now relaunches shell-only strands, which `batten-cancel-verb` meets — fixed: stated as a D1 consequence.
- board-fit LOW, `creel-agents` reply discipline — fixed: D4 says the note request expects no answer but `defer`.
- board-fit LOW, "prime" hard-codes orch's home against `orch-any-worktree` — rejected: that entry is an unbuilt note; the record states today's behaviour.
- board-fit LOW, watcher outside `runproc` — fixed: Constraints line.
- board-fit LOW, #811 split — fixed: Scope says #811 closes when both tasks land.
- existing-paths LOW, relaunch already guards the kill — fixed: D2 rationale names removal only.
- leanness MEDIUM, Notes for the plan writer duplicate `relevant.md` — fixed: section dropped, missing members added to `relevant.md`.
- leanness LOW, pass-only PATTERN constraints, auto-mode restating decisions, duplicate shell-test risk, condition-2 command strings, D7 narrative, Goal restating Scope, garbled D4 bound — fixed: all trimmed or reworded.

## Rejected alternatives

- A spawner-supplied engine process name on the strand, compared with `#{pane_current_command}`: a Node-based or version-named binary can report another name, and a false "dead" makes `start` remove a live orch.
- Checking that the pane's shell has no child process: Linux-only through `/proc`, while the tmux format is portable.
- Two readings some seconds apart instead of a launch grace: every status call would wait.
- Adopting a pane found in another session into reed: the operator decided reed never adopts a pane outside its session.
- Relaunching a dead orch strand from `status`: `status` would become a launcher; `start` owns that.
- Keeping the `written` reply as an alternative gate: two gates for one fact, and the reply is the cost the task removes.
- Withdrawing by deleting the marker without a lock: a withdraw could answer success for a cycle that started.
- Reclaiming in seatengine or shuttle instead of reed: every respawning caller (webster, Merge-In's conflict agent, burler rows) would need it.
- Reclaiming every dead strand of the same role, numbered forms included: wider than the reported failures need.
- Making shuttle stop wrapping `ErrSessionBusy` in its post-paste errors: other callers rely on retrying a draft hold; the mapping in creel's delivery is the narrower change.

## Review rounds

- Round 1: the open risk on Claude Code's tool commands staying off the tty was checked on this host with `ps -t` by the round-1 reviewer and again by its fixer.

## Question ledger

- Q1 liveness test — resolved, auto-pick: compare against the pane's shell; reed stays engine-neutral.
- Q2 boot window — resolved, auto-pick: 30-second launch grace; round 1 of the discussion review moved it from a recorded launch time to the age of the pane's first process, since old-binary launches record none.
- Q3 Windows — resolved, auto-pick: condition 3 skipped on Windows.
- Q4 watcher restart at status — resolved, auto-pick: live strand only, `start.lock` try-lock.
- Q5 note gate — resolved, auto-pick: turn end and non-empty note, from the phase's events offset.
- Q6 withdraw race — resolved, auto-pick: state lock shared with the cycle start; round 1 of the discussion review widened it to every watcher removal of the marker, and round 2 to every writer and remover of it.
- Q7 name reclaim — resolved, auto-pick: reed's add, exact name, not hidden, not live.
- Q8 #828 — resolved, auto-pick: post-paste errors map to `ErrNotLanded`; queued `/compact` withdrawn at the boundary; vanished worktree skipped.
- Q9 verb name — resolved, auto-pick: `lyx orch withdraw`; round 2 of the discussion review recorded that it never refuses, a departure from the board entry.
- Q10 hub rules section removal — resolved, auto-pick: out of the code change; the orch removes it with `lyx orch rules` after landing, as the section itself says.
