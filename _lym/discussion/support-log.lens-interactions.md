```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked every pair of decisions that share state, and each acceptance criterion and constraint against the decisions it touches.

**Findings**

1. **BLOCKING — D8 vs D3, D9 and D12: D8 refuses the hub's own programs.**
   - D8 refuses every sender inside the area that is in no pane.
   - D3 keeps several processes inside the domain that are in no pane: dobby, the reed watchdog daemon that posts D9's OOM notices, the background pusher and the loom and batten programs.
   - These processes post through the Go client as programs, or as `EXTERNAL`. `internal/creelengine/doc.go` says: "A process in no known pane is EXTERNAL … Through the Go client only, it can post as a program", and "The 0700 lymtemp root is the boundary."
   - So with D8 in force, the run watch, OOM notices and `<shortname>:LOOM` decision notices all fail.
   - Allowing them instead hits a second problem. D6 gives every agent write access to the area root's lymtemp, so any agent could post as `<shortname>:LOOM`, and agents obey that sender like the orch.
   - D8 needs a program identity that agents in the same domain cannot forge. Its acceptance criterion "a peer in the area and in no pane is refused" pins the breakage in place.
2. **BLOCKING — D15 vs D3, D7, D9, D12 and D13: inside detection fails on the AppArmor barrier.**
   - D3 decides inside or outside from `Seccomp_filters` plus the supervisor's pid.
   - With `barrier: apparmor` there is no seccomp filter, so every process inside reads as outside.
   - D7 would then refuse every lym verb inside the area.
   - The dobby and creel spawn sites would refuse.
   - D9, D12 and D13 would never route to the supervisor.
   - D15 says the supervisor "still serves D9, D12 and D13" but defines no inside test for that barrier.
3. **MEDIUM — D3 restart path: a dead server cannot be rebooted without `lym area stop`.**
   - The supervisor stays while any process in its tree lives: dobby, and the Slack relay (D12), which is its own child.
   - `lym area start` refuses while a supervisor runs, and reed boots an area hub's server only through `lym area start`.
   - So after the server exits with dobby alive, `lym reed attach` from the host names `lym area start` as the way forward, and that refuses.
   - The only way out is `lym area stop`, which ends every run. The record says this nowhere, and each refusal's way forward points at the other.
4. **MEDIUM — D3, D8 and D13 vs D6 and D10: locations in the per-user state directory are not fenced off from Landlock grants.**
   - Three things live in the per-user lym state directory:
     - the supervisor pid record (D3), which D3's inside/outside test and D8's supervisor-peer refusal rely on;
     - the deploy clone (D13), which the host builds from;
     - the area's engine config directories (D10), which D6 makes fully writable.
   - Nothing requires the first two to sit outside every granted path.
   - If they share a parent with D10's writable directory, an agent could forge the supervisor pid, or set `core.fsmonitor` or hooks in the clone, which runs on the host at the next deploy.
   - D2 states this exclusion for its own file; D3 and D13 lack it.
5. **MEDIUM — D12 vs PATTERN-child-processes: in-area paths cannot end the relay.**
   - The relay runs outside the Landlock domain as the supervisor's child, but it is a `runproc` HubOwner process.
   - Any in-area path that ends it through `runproc` gets `EPERM` from Landlock's signal scope; examples are a Slack toggle off or presence returning.
   - D12 gives the verb only a start and ends the relay only at `lym area stop`. The record does not say that those existing end paths break or must route to the supervisor.
6. **MEDIUM — D6 vs D8: the two sides may derive different socket paths.**
   - Inside, lymtemp builds its root from `TMPDIR` as `<area root>/lym-<uid>`, so the creel socket sits one level below the area root.
   - On the host, D6 says hubgeom and lymtemp derive "the area root" from the area file's presence, but it does not pin that both sides compute the identical socket path.
   - D6's acceptance criterion and D8's cross-hub criterion both depend on that match.
7. **LOW — D3 vs D8: another hub's worker cannot start this hub's creel daemon, and the record is silent.**
   - D3 refuses the creel daemon spawn from outside an area that is on. D8 covers only the in-area worker's side.
   - The record says nothing about the reverse direction: a hub without an area whose outbound worker tries to start the area hub's daemon.
   - That worker now gets a refusal, retries for an hour, and returns the post undelivered. It holds, but nothing in the record states it.
8. **LOW — Constraints: PATTERN-run-records is the wrong entry.**
   - The line "`PATTERN-run-records` and `PATTERN-child-processes`: `lym area stop` ends run processes only through `runproc`" cites `PATTERN-run-records`.
   - That entry governs friction, usage and project records, not ending processes. Only `PATTERN-child-processes` applies.

**Pairs checked**
- D3 × D7, host boot through `lym area start` and `lym reed attach`: held.
- D3 × D4, supervisor death to `ENOSYS` to stop and start: held.
- D3 × D3, restart with a live supervisor: broken (finding 3).
- D3 × D8 and D9 and D12, detached processes in no pane: broken (finding 1).
- D3 × D15, inside detection: broken (finding 2).
- D3 and D13 × D6 and D10, write grants on the state directory: broken (finding 4).
- D6 × D8, socket derivation: unpinned (finding 6).
- D6 × D10, `CLAUDE_CODE_TMPDIR` and config directories: held.
- D6 × D11, read grant on the token file: held.
- D2 × D6 and D7, area file read only from outside: held.
- D4 × D8, `SO_PEERCRED` naming the supervisor, handled by `SCM_CREDENTIALS`: held.
- D4 × D9, D12 and D13, supervisor socket connect and verbs bounded by target, not caller: held.
- D5 × D9 and D13, supervisor socket on the derived list: held.
- D7 × D8, other hubs' daemons run with their own hub as cwd: held.
- D7 × D13, `window` runs `lym reed attach`, which D7 allows; `deploy` runs with the clone as cwd: held.
- D8 × D3, another hub's worker spawning this hub's daemon: held, unstated (finding 7).
- D9 × D3, panes descend from the supervisor: held.
- D12 × PATTERN-child-processes, runproc end of the relay: broken (finding 5).
- D13 × D2, deploy command from the area file: held.
- D14 × D8, creel identity landed before the boot: held.
- D16 × D7, operator works only from the server's shell window: held.
- Acceptance D8 "a peer in no pane is refused": contradicts finding 1.
- Acceptance D6 and D8 cross-hub: rest on finding 6.
- Constraint PATTERN-run-records: miscited (finding 8).

Further tool calls:
- Read `internal/creelengine/doc.go` lines 38 to 60, how creel identifies a program sender. My context held only the doc's grep lines about `EXTERNAL`, not the program-name rule.
