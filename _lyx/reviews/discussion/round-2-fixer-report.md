# Fixer report — discussion, round 2

## Changes

- F1 (BLOCKING): D5 gains a bullet: a requested cycle's handoff timeout in `tickHandoff` measures from its start, `CycleRequestedAt` when that predates `PhaseEnteredAt`, less the hub pause coverage from the same `CoverageOf` intervals. The acceptance criterion now asks that a request held by a pause longer than `handoff_timeout_s` runs its cycle to completion after the resume.
- F2 (MEDIUM): D1 condition 2 now states today's fact. `validateDisplay` admits only own-window, overview and hidden, retired anchors decode as own-window, and a hidden strand owns no pane. So `presentPanes` already reads every bound pane session-wide and reconcile already keeps its binding. The widening, reconcile and render sentences are deleted. The shuttle-stop consequence drops "in the shared window", and the shared-window acceptance criterion is now about an own-window strand moved to another window of the session. Fixed differently in one point: the consequence's claim that the stop would otherwise leave the pane was false, since `strandRemovable` already takes a present own-window strand, so the bullet says D1 changes nothing there.
- F3 (MEDIUM): D2 takes the first option: reconcile keeps the binding of a strand whose pane the server-wide lookup finds in another session. D2 records that the #746 `%3 0:bash bash` came from a `tmux list-panes` on an unnamed server, and that pane ids are per server while reed runs its own `-L` socket. Its bound adds that a pane in another tmux server is not found. The acceptance criterion requires `strand_elsewhere` after a reconciling reed op.
- F4 (MEDIUM): D5's serialization bullet adds that `RequestCycle` writes the marker under the state lock by a temp-file rename. `requestCycle`'s no-watcher removal becomes a compare-and-remove under the same lock, so every writer and remover holds it.
- F5 (MEDIUM): fixed by the second option. D5 keeps the never-refuse answer and records it as a departure from the board entry, with its reason: the answer already names the running cycle, and a refusal would make an unavoidable lost race read as a failed command. The Auto-mode assumptions list it too.
- F6 (LOW): D1's grace reads the pane pid's start time from `/proc/<pid>/stat` field 22 against `/proc/uptime`, as `proctree_linux.go` reads `/proc`, so a read spawns no process. Fixed differently in one point: the proposal says "against the boot time", but `/proc/uptime` gives the same age with no boot-time read.
- F7 (LOW): D7 states the bound: a not-a-git-repository resolve is skipped whatever its cause, and its strands get no delivery until it resolves again. The logging change was revised in the self-check, see below.
- F8 (LOW): D5's distill and refresh bullet adds `held_by_pause` and the note only when a live watcher kept the request. With no live watcher, the verbs answer as today.
- F9 (LOW): researched in the code and recorded in D6's rationale.
  - A burler fix attempt runs as its table's chair (`fix.go`), and `seatSpec` gives the chair `KeepPane` false.
  - Every end of a started chair removes its strand: `done` through shuttle's finalize, a timeout or wait error through `stopSeat` in `join`, and `died` through the died-window removal.
  - So the #811 holder was a registration those removals skipped (`removeStrandIfPresent` skips a strand neither live nor present) or one whose removal failed.
  - A failed removal can leave a live holder, which `checkName` still rightly refuses, so D6 adds no teardown.
- F10 (LOW): the tmux fallback chain is kept once, in Constraints, and D1's bullet is reduced to its consequence. The first open risk keeps the version and `ps -t` evidence, and its review-round narrative moved to support-log.md's Review rounds.
- F11 (NIT): D3's bound now names two `status` calls in sequence, the second one before the first one's watcher takes `watch.lock`.

## Self-check

- fix-way-forward, MEDIUM: the D2 criterion's step, `lyx orch start`, removes the dead orch strand through `StartRelaunch`, so it cannot show `strand_elsewhere` afterwards. **Fixed as proposed**: D2 says start's relaunch drops the binding, and the criterion names a reconciling op that does not remove the strand, another strand's add.
- fix-claims, MEDIUM: D6 said a failed removal leaves the pane live, but a removal can fail after the pane went. **Fixed as proposed**: a failed removal leaves either a live pane that `checkName` refuses, or a dead registration that D6 reclaims.
- fix-claims, LOW: "a failed removal logs a Warn and goes on" holds only for finalize and died-window removals. **Fixed as proposed**: D6 adds that a failed `stopSeat` fails the attempt with a `lyx reed remove` way forward.
- fix-consistency, LOW: the shuttle-stop consequence disagreed with the removal list. **Fixed as proposed**: the bullet says D1 changes nothing for the stop, and shuttle's stop leaves the list of removals that read the judgment.
- fix-consistency, LOW: the `/proc` read is Linux-only while two criteria said "On POSIX". **Fixed as proposed**: both now say "On Linux".
- fix-consistency, LOW: ledger entry Q6 was stale against the widened lock. **Fixed as proposed**: Q6 names round 2's widening to every writer and remover.
- fix-consistency, LOW: ledger entry Q9 omitted the never-refuse departure. **Fixed as proposed**: Q9 records it.
- fix-consistency, LOW: relevant.md misses `planReconcile`, `tickHandoff` and `proctree_linux.go`, and still calls `launchStrandLocked` the place to record a launch time. **Deferred**: relevant.md is outside this round's write surface.
- fix-proposal, MEDIUM: after a reboot, a kept binding can match a sibling worktree's fresh pane of the same id on the shared hub server, which misnames the placement. **Fixed with my own change**: D2's bound states the misreport and its limits. The strand still reads dead, and the misreport lasts only until a removal drops the binding or that pane goes. I made no lookup that tells a move from a reused id, because reed records nothing on a pane that would tell them apart: its tmux options mark windows and sessions, not strands.
- fix-proposal, LOW: a Warn on every routine staging-worktree teardown hides the signal it is meant to give. **Fixed as proposed**: Info when the directory or its gitdir is gone, Warn when both still exist.
- fix-proposal, LOW: the tick-to-seconds conversion was unstated. **Fixed as proposed**: D1 names `USER_HZ` from `sysconf(_SC_CLK_TCK)`, read through cgo, which the `lyx` build already needs.

## Disputed

None.

## Deferred

- fix-consistency LOW, `_lyx/discussion/relevant.md` stale against D1, D2 and D5: outside this round's write surface (the decision record, the support log and this report), so the next author of relevant.md has to add `planReconcile`, `tickHandoff` and `proctree_linux.go` and drop the launch-time note on `launchStrandLocked`.
