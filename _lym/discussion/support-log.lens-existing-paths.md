```yaml
lens: existing-paths
outcome: reported
```

Lens existing-paths: what each decision runs into in the code that already runs.

**Findings**

1. MEDIUM — D2 and D3: the merged Stop command reads its standard input twice, but stdin can only be read once.
   - `recordingCommand` writes the Stop payload to `events.jsonl` with `cat >> <events>`, so it consumes stdin.
   - D3 has `hook-read` read the hook's stdin to check `agent_id`, and D2 runs `hook-read` before the recording, in the same command.
   - So whichever runs first takes the payload. If `hook-read` goes first, the turn-end record loses its payload: `ParseEvents` needs `hook_event_name`, `last_assistant_message` and `background_tasks` from it, and without them it reads no turn end.
   - The record says nothing about this. The fix is a decision that the merged command saves stdin once and passes it to both `hook-read` and the recording.

2. MEDIUM — D4 and D7: the waiter's daemon connection drops every time the daemon restarts, and the record doesn't say what the waiter does then.
   - The daemon ends whenever its binary is replaced, which happens on every deploy, and closes its listener. Every waiter's connection drops with it, and the waiter registrations, held only in daemon memory, are lost.
   - Today a dropped waiting call reconnects up to `MaxReconnects` times (4) and resumes the same wait.
   - D4 counts a waiter live only while its connection is open, but never says whether `hook-wait` reconnects and registers again, or exits. If it exits, a deploy leaves every idle strand without a waiter until its next `Stop`, and D5's typed bell becomes the normal path.
   - Fix: state that the waiter reconnects and registers again, or accept the D5 fallback after a restart and say so as a bound.

3. MEDIUM — D4 and D5: a wake turn is invisible to shuttle's "a turn started after the boundary" check.
   - `Run.boundaryIdle` and `turnStartHold` in `internal/shuttleengine/send.go` spot a new turn only by an unmatched `UserPromptSubmit` signal.
   - An asyncRewake turn submits no prompt, so it records no `UserPromptSubmit`; D5's typed bell does record one.
   - So after a producer's turn ends with mail standing, D4 can start a wake turn while `boundaryIdle` still says the session is idle, and the gate goes ahead with its re-prompt send into a turn that is running.
   - Turns that fork completions start already behave this way, and the guarded paste-submit probably refuses to type into a busy pane. But D4 makes this happen at every boundary where mail is waiting, and the record doesn't mention it.
   - Fix: state the bound, or let `boundaryIdle` also read the creel ring.

4. LOW — D2 and D4: a woken turn's `Stop` gets recorded with no turn start before it.
   - After a ring, the wake turn ends and records a turn end that shuttle never prompted.
   - The run loop can then judge the run's outputs again, or re-prompt, at a boundary it didn't cause.
   - A typed notice caused the same thing before, so this is not new behaviour. But the record should say that D4's ring keeps it rather than leave it implied.

5. LOW — D6: old journal records still need to fold.
   - `fold.go`, `journal.go` and `deliver.go` carry the `reminded` records and the notice reservations, and `log --id` lists `reminded` events.
   - The record's Constraints already cover this under `PATTERN-tolerant-records`. Old records still fold and `log` still lists them, so this holds.

**Paths traced, per decision**

- **D1:**
  - The old `PostToolUse` hook of sessions launched before the deploy calls the new `hook-read` and gets the block shape; it still works. Survives.
  - `ClaimAtBoundary` budget: `MessageLine` grows by two quote characters. Survives.
- **D2:**
  - `recordingCommand` reading stdin: breaks (finding 1).
  - `ParseEvents` and `ParseSessionSignals` in `shuttleengine` and `websterengine` while a Stop is blocked: no record, so they read right. Survives.
  - `eventsFileActivity` and `newestTurnEnd`: survive.
  - Exit status of the Stop recording: survives as the record states it.
- **D3:**
  - Old sessions' hooks inherit stdin, so the check still gets its input. Survives.
- **D4:**
  - Daemon restart on binary replacement: breaks (finding 2).
  - `boundaryIdle` and `turnStartHold`: misread (finding 3).
  - The idle reading through `hubDelivery.Idle` and `SessionIdle(capture)`: reads the pane, so it still reads right. Survives.
  - Waiter identity through the pane process tree: the hook is Claude's child, inside the pane tree. Survives.
- **D5:**
  - The guarded send, busy requeue and pause rules: survive.
  - The in-memory "already belled" set after a daemon restart: the record notes it. Survives.
- **D6:**
  - `deliver.go`'s notice send and `markNoticeRead` reservation go away; no other path relies on them. Survives.
  - The journal fold: survives.
- **D7:**
  - Under a pause, the interrupt claims the strand into the pause, and `lym tender resume` types the resume line; the turn's `Stop` then delivers the mail. Survives.
- **D8:**
  - The parent-directive and webster stencils match `[creel]` lines as text a model reads; no Go code parses the sender. Survives.
  - `orch-template-role.md` and `orchcli/doc.go` are covered by D11.
- **D9:**
  - `--append-system-prompt` is built only in `claudeengine` `buildLaunchCmd`, on the launch and resume lines. Survives.
  - A resume line is never rebuilt, so sessions launched before the deploy lack the sentence; the Open risks cover it.

**Tool calls**

1. Searched for readers of `MessageLine` and the creel line format, the callers of `ParseSessionSignals`, and the notice and reminder files: my context lacked the reader list.
2. Read `signals.go`'s header, `activity.go`'s event-time readers and the idle-check entry points: my context lacked how activity and idle are read.
3. Read `recordingCommand`'s body, the client's reconnect rule, the `Strand` type and `boundaryIdle`: my context lacked how stdin is used, how reconnect works and how a turn start is detected.
