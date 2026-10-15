---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D1, D4 and Acceptance criteria
    summary: The D1 hint sends the operator to --continue, but D4 has the plain fresh launch overwrite the recorded id with its own new id first, so the hint resumes the session just launched and the old conversation's id is lost
    fix: In decision-record.md D1, change the hint to name `lyx orch stop` then `lyx orch start --adopt <id>` with the id the state recorded before this launch written out inline, read before the launch's save replaces it, and say in D1 that this hint is the one place the previous id survives once D4's write has run. Change the Acceptance criterion "A plain fresh launch with a recorded id names `--continue` in its hint" to "A plain fresh launch with a recorded id names `lyx orch stop` then `lyx orch start --adopt <previous id>` in its hint, with that previous id; with none recorded, no such hint". In Scope's In list, change "names `--continue` in its envelope hint" to "names the previous id's `--adopt` command in its envelope hint".
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md Scope Out (reed resume bullet), D4 and Open risks
    summary: After a reed server rebirth the reborn strand replays --resume of the launch id, its session-start signal names the pre-clear id, and the watcher a later lyx orch start spawns writes that id over the post-clear one, so the Out bullet's claim that lyx orch stop then --continue recovers the current session is false once a watcher has ticked
    fix: In decision-record.md, replace the Out bullet's sentence "A strand an explicit `lyx reed resume` relaunched is live, so `--continue` refuses it; `lyx orch stop` then `--continue` recovers the current session." with "A strand an explicit `lyx reed resume` relaunched is live, so `--continue` refuses it; `lyx orch stop` then `--continue` recovers the post-clear session only while no watcher has ticked on the reborn strand, since the watcher records the replayed pre-clear id." Add an Open risks bullet stating that a watcher bound to a strand reed's rebirth replayed records the pre-clear id, the post-clear id is then recoverable only from the provider's own session list for `--adopt`, and `lyx orch status` shows the pre-clear id.
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D3 Bound and D4
    summary: D4 names only fresh launches and the watcher as writers, but start's live-strand branch rebinds the state onto any live strand named orch without a fresh launch, leaving session_id naming the previous strand's session while Strand names the new one, and the D3 Bound does not name this rebind as a route by which the record gets written
    fix: In decision-record.md D4, add after "`start` writes it at every fresh launch, plain or adopt, with the id the launch used;" the sentence "when `start` binds the state to a live strand other than the recorded one without launching, it clears the id, and the watcher's first reading of that strand's run fills it." In D3's Bound, replace "a session orch did not launch or adopt" with "a session other than one orch launched, adopted with `--adopt`, or bound as its live orchestrator strand". Add an Acceptance criterion "`start` binding a live strand other than the recorded one clears the recorded id until the watcher reads that strand's id; a strand with no shuttle run leaves it empty, so `--continue` refuses".
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4 and Open risks
    summary: D4 takes the id from the newest signal of any kind, so a subagent or fork hook line naming its own session id would land the record on a foreign session; the only bound, that Claude Code sends the parent id there today, lives only in the support log, which Plan-Write never reads
    fix: In decision-record.md Open risks, add the bullet "D4 reads the id from the newest signal of any kind, relying on Claude Code naming the parent session id in a subagent's or fork's hook lines; a version that names the subagent's own id there makes the watcher record it, and `--continue` then resumes the subagent's session."
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D1 and Acceptance criteria
    summary: A relaunch while the old watcher is still stopping already sets the envelope hint to the watcher-stopping text, which would replace the D1 hint, and D1 does not say which hint a launch carrying both reports
    fix: In decision-record.md D1, add the sentence "When the watcher-stopping hint also applies, the envelope's hint carries both texts, the watcher-stopping one first." Extend the D1 hint Acceptance criterion with "including a launch that also reports `watcher_stopping`".
  - id: F6
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints, docs bullet
    summary: The Constraints bullet saying docs land in the same commit repeats Scope's In bullet naming the help text and both doc.go files, and the repo's own CLAUDE.md rule
    fix: "Delete the Constraints bullet that begins \"Docs land in the same commit: the flag and the record go into the package `doc.go` files\"."
---

### [BLOCKING] The D1 hint points at a session id that D4 has already overwritten

**Location:** `_lyx/discussion/decision-record.md` D1, D4, Acceptance criteria.

**Issue:** D1 says that when the state records a session id, a plain fresh launch's hint names `lyx orch stop` then `lyx orch start --continue` "as the way back to that conversation".
D4 says `start` writes the id "at every fresh launch, plain or adopt, with the id the launch used", and D3 says only a failed launch writes no state.
So by the time the operator reads the hint, the successful plain launch has replaced the recorded id with the fresh session's minted id.
Following the hint stops the fresh session and resumes that same fresh session.
The old conversation's id is gone from orch state, and `lyx orch status` (D6) cannot show it either.
The Goal-scope lens fix in the support log (Turn 5, Q7) therefore does not do what it claims, and the matching Acceptance criterion pins the broken path.
The read path is confirmed in `internal/orchcli/start.go`: the state is loaded before the launch and saved through `ResetForFreshLaunch` after it, so the previous value is available to a hint only if it is captured before that save.

### [MEDIUM] Reed rebirth makes the watcher regress the recorded id to the pre-clear session

**Location:** `_lyx/discussion/decision-record.md` Scope Out, D4, Open risks.

**Issue:** `internal/orchengine/doc.go`'s residual and `internal/reedengine/strand.go` confirm that reed's resume replays the strand's stored `ResumeCmd`, which carries the launch's `--resume <id>`, on the same strand.
After a clear-mode cycle, the reborn session is the pre-clear one, and its session-start hook line lands in the same run's events file.
D4's rule (newest signal naming an id, written when it differs) then records the pre-clear id on the first tick of the watcher that `lyx orch start` spawns for the live strand.
The Out bullet tells the operator that `lyx orch stop` then `--continue` recovers the current session, but after that tick it resumes the pre-clear one again.
This is the regression case focus entry 1 asks about, and the record states no bound or criterion for it.

### [MEDIUM] The live-strand rebind writes the record without a stated rule

**Location:** `_lyx/discussion/decision-record.md` D3 Bound, D4.

**Issue:** `start`'s `StartSpawnWatcher` and already-running branches call `adoptStrand`, which binds the state to any live strand addressed as the orchestrator whose guid differs from the recorded one, through `ResetForFreshLaunch`, without launching anything.
D4 defines `session_id` as the id of the strand `State.Strand` names, but names only fresh launches and the watcher as writers.
Under the record as written, the rebind keeps the previous strand's id beside the new strand's guid.
When that strand has no shuttle run, `Runner.SessionState` fails on `FindRun`, so the watcher never corrects it.
The rebind is also a third route by which the record gets written, and D3's Bound, "a session orch did not launch or adopt", does not say whether a rebind counts as adoption.
That leaves focus entry 2's question open.

### [LOW] The fork and subagent hook-id assumption lives only in the support log

**Location:** `_lyx/discussion/decision-record.md` D4, Open risks.

**Issue:** `internal/shuttleengine/claudeengine/signals.go` gives every signal kind, `PreToolUse` included, the payload's `session_id`, and `readSessionFacts` takes the newest signal's id regardless of kind.
The orch spec sets `ForkSubagents`.
A subagent's or fork's hook line naming its own id would make the watcher record that foreign session.
The support log rejects this on the grounds that Claude Code sends the parent id today.
That is a provider-internal dependency of the same kind as the `/clear` risk the record already lists, but the plan writer never sees it.

### [LOW] Two envelope hints can collide

**Location:** `_lyx/discussion/decision-record.md` D1, Acceptance criteria.

**Issue:** `start.go` sets `fields["hint"]` to the watcher-stopping text whenever the watcher is still live and stopping, and a dead strand relaunches whatever the watcher's state.
A plain fresh launch with a recorded id and a stopping watcher has two hints competing for one envelope key, and D1 does not say which one wins.

### [NIT] The docs constraint duplicates Scope

**Location:** `_lyx/discussion/decision-record.md` Constraints.

**Issue:** Scope's last In bullet already names the `start` help text and both `doc.go` files, and CLAUDE.md states the same-commit rule for the repo.
The Constraints bullet adds nothing the plan writer would act on.

## Rejected

- D3's guard set is complete against `start.go`: the `--adopt`/`--handoff` exclusivity, the live-strand refusal and the shuttle resume check all exist, and `validateSessionID` applies the UUID shape guard to any id `--continue` passes through `ResumeSessionID`.
- A quiet tick cannot regress the id to the launch id: `readRunSessionState` reads every signal past the prompt offset on each call, and the prompt offset is set only once, at launch (`run.go`).
- The watcher's strand-checked save (`saveStateForStrand`) refuses to write onto a strand a concurrent `start` bound, as D4 claims.
- The `creel-stage1-fixes` overlap is carried in Constraints, and the `depends_on` call is the orchestrator's, not the plan writer's, so it needs no place in the record.
- No Decision edits the task's own board entry.
- D2's `session_id` envelope field and D6's status line are read-only outputs and open no attack surface.
