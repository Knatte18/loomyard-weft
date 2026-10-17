# Fixer report — discussion round 1

## Changes

Every finding's premise held against the code; all are fixed in `_lyx/discussion/decision-record.md`, with the support log's Q3, Q7, Q9, Q15, Turn 4 and a round-1 entry updated to match.

- F1 (fixed as proposed): D5 Restart persists the pending step's persist time and adopts only an inject since then whose command name and argument, or text, match; the persist time tells skills from retry. The restart criterion covers each reload step, the pointer and a retry of every skill.
- F2 (fixed with the proposal's first option): D5 gains a Fresh launch bullet. `lyx orch start` withdraws ORCHWATCH's queued injects and releases its hold, through a new Go-client-only `withdraw` verb; Scope In names the verb. A criterion covers a mid-cycle relaunch. The GUID-binding alternative was not taken, since creel addresses and keys queues by strand name (`internal/creelengine/seam.go`), so a binding would change every inject.
- F3 (fixed as proposed): `unconfirmed` advances the color and plugins steps and the bind color; those steps get the handoff timeout; the Hold bound names it. Criteria cover an unconfirmed and a repeatedly expiring plugins inject.
- F4 (fixed with the proposal's first option): D4 persists the gate's outcome inside the Go client's read handler, which runs before the ack (`internal/creelengine/client.go`, `handle(resp)` before `ackLine`). The journal option was not taken: `Log` returns no body and no `reply_to` (`verbs.go:320`, `fold.go:108`). A restart criterion covers it.
- F5 (fixed as proposed): D5's transcript bullet reads only events after creel's `sending` record; criterion added.
- F6 (fixed as proposed): the hold is taken on entering any non-idle phase, the auto-compaction reload included; criterion extended.
- F7 (fixed as proposed): D6 signs as the strand only from that strand's pane, else as the program with no strand claim; the sender criterion names a detached `hibernate watch`.
- F8 (fixed with the proposal's second option): D1's bound names the sender-resolution refusal; D5 holds the cycle and shows it as State.Stuck; the creel-failure criterion covers it.
- F9 (fixed as proposed): D6's wake-up reads with the command the reminder names, and the stencil says so.
- F10 (fixed as proposed): D6's bound covers both signers; D8's fallback reads `lyx creel log --to <orch strand>`.
- F11 (fixed as proposed): D4's answer is quoted.
- F12 (fixed with the proposal's second option): D9's pause-resume line goes as ORCHWATCH mail, marked noticed only after the post succeeds; D5's inject list and the pause criterion follow.
- F13 (fixed differently): D7 injects as `TENDER` with no strand claim. No new refusal was added: `tendercli`'s `refuseCaller` (`internal/tendercli/cli.go:56-61`) already refuses a hub-scope revive from an agent's pane with a way forward, so D7 states that.
- F14 (fixed with the proposal's second option): the orch revive's lookup and inject run inside a `pauserecord.Update`, and D7 states the lock bound.
- F15 (fixed as proposed): Scope In and D7 state the report change and `tenderengine`'s doc.
- F16 (fixed as proposed): D4 and D6 resolve the orch strand's name from its GUID through reed at each call.
- F17 (fixed as proposed): the Scope In clause is replaced with the state keys and the status help text.
- F18 (fixed as proposed): D2's reason points at stage 3's reply discipline; Q3 updated.

## Self-check

- fix-consistency, MEDIUM, D5's opening said the timeouts and persisted phases stay as they are: fixed as proposed; it now excepts the reload timeouts and the persist time.
- fix-consistency, MEDIUM, State.Stuck descriptions omitted the creel failure and refusal: fixed as proposed in D5 and Scope In.
- fix-consistency, LOW, the Scope `withdraw` bullet named one use only: fixed as proposed.
- fix-consistency, LOW, support log Q7 and Turn 4 were stale: fixed as proposed.
- fix-consistency (outside its lens), skills, retry and pointer timeouts ran from first typing, so a never-typed step never timed out: fixed as proposed; every reload step's timeout counts from first queueing.
- fix-proposal and fix-way-forward (merged), MEDIUM, the withdraw ran after `StartSession` (`start.go:296` before `:304`), and a live old watcher could requeue: fixed as proposed. Start withdraws before starting or binding and again after the save. A `withdrawn` expiry queues nothing and returns to idle. A watcher queues only after re-reading the state on that tick.
- fix-way-forward, LOW, the first queueing time was not persisted across requeues and restarts: fixed as proposed in D5 Restart.
- fix-proposal and fix-claims (merged), LOW, the orch revive is not inside `pauserecord.Update` today (`resume.go:295-299`), and the lock bound went stale: fixed as proposed; D7 moves it there and restates the bound.

## Disputed

None.

## Deferred

None.
