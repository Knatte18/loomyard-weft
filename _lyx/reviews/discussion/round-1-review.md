---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:186
    summary: D11's skip rule keys on the recording command's session-start event, which is written even when resume-context fails, so a failed verb leaves neither the hook nor the typed fallback delivering the pointer, contradicting D11's own Bound
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:183
    summary: resume-context's prompt choice is not fully determined by the persisted phase; a watcher tick that reads the boundary before the hook runs moves the phase to resuming, the verb renders the reload prompt without the handoff note, and the watcher then skips its own pointer
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:104
    summary: D5's premise that every lyx-booted session comes through a verb that spawns the watchdog child is false; orch start, loom run and step, and batten's task-worktree resume boot sessions through reedengine without a watchdog spawn, so they get no discover signal
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:103
    summary: D5 adds the discover-signal routing edge with no Bound; any writer to the hub scratch dir can force a list-sessions cycle per touch with no rate limit, and the record never states what bounds that
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:129
    summary: D7 holds on an unmatched turn start with no release, while the prime's orch events file shows turn starts without a turn end are common and the send's existing turnStartHold release rests on the pane D7 stops trusting
  - id: F6
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:128
    summary: D7's ready and held lists do not cover the fold's whole state set (unknown with causes other than unreadable events, dead), and its literal newest-signal-is-a-turn-end test fails once D11's session-start signal follows the turn end
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:103
    summary: D5 says the discovery loop consumes the discover signal and runs a cycle, but D3 delivers remove events too, so without D4's stat-first rule the consumption's own remove event triggers a second cycle and reset
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:264
    summary: The plan-writer note says IdleProbe can grow a Reason field rather than be replaced, while D7 says SessionIdle returns the new reading in place of IdleProbe
  - id: F9
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:223
    summary: The fsnotify v1.9.0 pin appears only as an auto-mode assumption, not in D3 or Constraints where the plan writer reads requirements
  - id: F10
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:228
    summary: Open risks restate D7's Bound, D5's three-minute reap and the hook-semantics assumption nearly verbatim; keep only the residual risk each adds
---

### [BLOCKING] D11's fallback is defeated by the very event that is meant to confirm delivery

**Location:** `_lyx/discussion/decision-record.md:186`, with the Bound at `:190`–`:191` and the acceptance criterion at `:248`.

**Issue:**
The session-start hook runs two independent commands: the recording command and `lyx orch resume-context`.
The watcher skips its pointer step when it reads "a `session-start` event with source compaction" after the boundary.
That event comes from the recording command, which appends the payload whatever the verb does.
D11's Bound says "A verb failure prints its JSON error, which Claude Code shows and does not add as context, and the typed fallback delivers the pointer."
Under the skip rule as written it does not.
The event is recorded, the watcher skips the pointer, and the session gets no pointer at all.
A stencil render failure, a missing `State`, or a cwd that does not resolve to the prime all hit this.
The draft-case acceptance criterion repeats the flaw, since it takes the `session-start` event as proof that "the resume pointer is in the session's context".
The event shows only that the hook fired.

The reverse window is not bounded either.
If the event has not been read by the time the plugins step is done, the pointer is delivered twice.
That is harmless, but the record should say so.

**Fix:**
Base the skip on evidence that the verb succeeded, not on the hook firing.
For example, the verb records its own successful render (a mark in orch state or the events file) and the watcher skips only on that mark.
Alternatively, make the verb the only recorder of the confirming event.
State in D11 that a failed render leaves the mark unset, so the typed pointer follows.
State that a late-read event can make the pointer arrive twice, which is harmless.
Reword the acceptance criterion to prove delivery by that mark.

### [BLOCKING] The resume-or-reload choice is not settled by the persisted phase alone

**Location:** `_lyx/discussion/decision-record.md:183`, and the Note at `:267`.

**Issue:**
D11 has the verb render the resume prompt naming `State.LastHandoff` "when the persisted phase is `compacting` and the reload prompt naming the role alone otherwise."
In the code, `tickCompacting` completes on the boundary found in the transcript once the probe passes (`internal/orchengine/watcher.go`, `tickCompacting` → `reloadAfterCompaction` → `startReload` → `enter(PhaseResuming)`).
That does not wait for the session-start hook.
Under D7 the readiness after a `/compact` reads ready at once.
`/compact` fires no `UserPromptSubmit`: the prime's orch events file records `/distill` prompts but no `/compact`.
So a watcher tick can land between the boundary write and the verb's state read, and the verb is a fresh `lyx` process.
In that window the phase is already `resuming`, the verb renders the reload prompt without the note, and the cycle's handoff is lost.
The recorded `session-start` event then makes the watcher skip its own pointer, which held the right resume prompt in `PendingResume`.
A second, smaller gap: an automatic compaction inside `handoff_requested` or `resuming` gets the reload prompt from the hook.
The watcher types no pointer in those phases, since `startAutoReload` runs from `tickIdle` only.
So "the pointer the watcher would type" does not hold there.

**Fix:**
Have D11 state the choice over the full state.
For example: `compacting`, or `resuming` with `ReloadSkipsSkills` and a `PendingResume` naming a handoff, renders that resume prompt, and anything else renders the reload prompt.
Or have the verb read `PendingResume` when it is set.
Also state what the hook delivers when auto-compaction happens in a non-idle phase, and why that is acceptable.

### [BLOCKING] D5's discovery premise is false

**Location:** `_lyx/discussion/decision-record.md:104`, and the assumption at `:222`.

**Issue:**
D5 says "every `up`, `resume` and `attach` already spawns that child, so a session booted by lyx is discovered within a cycle of its boot with no new edge in the verbs."
`SpawnWatchdog` is called from `reedcli`'s `up`, `resume` and `attach` (`ensureWatchdogSpawned`), and from `loomcli/start.go:547`.
Other lyx paths boot the session through `reedengine` without it:

- `orchcli`'s `reedUp` (`internal/orchcli/cli.go:150`);
- `loomcli`'s `run` and `step` bootstrap (`internal/loomcli/arm.go:331`, `:476`);
- battencli's revival of a task worktree via `reedengine.New(...).Resume()` (`internal/battencli/wire.go:791`).

Those sessions get no discover signal.
They are found only at the backed-off cadence, up to 60 s, and their resize loop does not start until then.
The auto-mode assumption confines that delay to "a session created by hand outside lyx verbs", which understates it: the orch session and batten-revived task worktrees fall in the same bucket.

**Fix:**
Correct the premise.
Then either touch the discover signal at the boot chokepoint: in `reedengine` where a session is created, or by calling `SpawnWatchdog` from the sites above.
Or accept the up-to-60 s delay for those paths and say so in D5 and the assumption, naming the paths as a group.
The rubric bars enumerating call sites, so the Decision should name the boot path, not a list.

### [MEDIUM] The discover signal has no stated bound

**Location:** `_lyx/discussion/decision-record.md:103`.

**Issue:**
D5 introduces a routing edge.
A file under `fabricengine.HubScratchDir(hub)` wakes the daemon "at once", runs a cycle and resets the cadence to 5 s.
Unlike D2, D7, D9, D10 and D11, D5 has no `Bound:` line.
Nothing limits how often a touch runs a cycle, so a toucher in a loop drives `list-sessions` faster than today's 5 s base.
The record never says who may write the file: the scratch dir is writable by any process of the operator.

**Fix:**
Add a Bound to D5.
State who writes the signal: the lock-held watchdog child, at most once per verb invocation.
State that a wake is consumed, and that wakes run no faster than the base cadence (5 s) or another floor the loop names.
State that the worst case is therefore today's cost.

### [MEDIUM] D7's unmatched-turn-start hold has no release

**Location:** `_lyx/discussion/decision-record.md:129`, and the acceptance criteria at `:246` and `:253`.

**Issue:**
D7 holds on "an unmatched turn start" and states no way out of the hold.
The prime's orch events file (`.lyx/shuttle/b1b9d1b7…/events.jsonl`) records 172 `UserPromptSubmit` lines against 144 `Stop` lines.
Turn starts without a turn end are routine: interrupts, and turns that end without Stop.
The send's existing `turnStartHold` (`internal/shuttleengine/send.go`) releases such a start two ways.
One is an interrupt marker through `SessionProber`.
The other is `turnStartIdleOverride` (10 s) of an idle pane.
The second rests on the pane idle reading that D7 stops treating as authoritative.
D7 brings the same hold into the watcher, which has no such hold today.
It does not say whether the watcher uses `turnStartHold`, or what releases a start the interrupt marker misses.
An unreleased start holds every injection until the operator's next turn end, the #481 symptom in a new form.

**Fix:**
State the release in D7.
For example, the readiness reading carries `turnStartHold`'s interrupt release and its idle override, with the override's pane condition named.
Add the release to the acceptance criteria.

### [MEDIUM] D7's state classification is not exhaustive and conflicts with D11's new signal

**Location:** `_lyx/discussion/decision-record.md:128`–`:131`, against the acceptance criterion at `:246`.

**Issue:**
The fold (`internal/shuttleengine/sessionstate.go`) also yields these states:

- `unknown`, with cause `liveness-unproven`, `no-signal`, `no-state`, `contradiction` or a session end's reason;
- `dead`, with cause `process-gone`.

D7 places only "an unreadable session state" (falls back to the pane probe).
Every other `unknown` and `dead` is in neither the ready nor the held list, and the acceptance criteria repeat the gap.
A freshly bound orch session reads `unknown`/`no-signal` until its first hook, and that is the state `typeStartColor`'s gate meets.
Separately, Ready opens with "the newest hook signal is a turn end".
D11 adds a `session-start` signal that lands after the turn end at every compaction.
Read literally, the post-compaction reading is not ready.
The fold leaves the derived state unchanged on an unknown kind, so the two halves of the Ready condition disagree on the exact tick D11 targets.

**Fix:**
Make D7's lists cover every state name, with a rule for `unknown` and `dead` (pane fallback, or held with a reason).
State the Ready condition over the folded state rather than the newest signal, or say that a `session-start` signal leaves readiness unchanged.
Mirror both in the acceptance criteria.

### [LOW] Consuming the discover signal can wake the loop a second time

**Location:** `_lyx/discussion/decision-record.md:103`.

**Issue:**
D3 delivers an event per create, write, rename or remove of a named entry.
D5 has the loop "consume" the signal (remove it), run a cycle and reset the cadence.
D4 handles this with "stat, remove", so a remove event finds nothing.
D5 states no such rule.
Each touch can therefore produce two cycles and two resets, the second triggered by the loop's own removal.

**Fix:**
State in D5 that a wake whose stat finds no signal file runs no cycle, as D4 does for the resize signal.

### [LOW] The plan-writer note contradicts D7 on `IdleProbe`

**Location:** `_lyx/discussion/decision-record.md:264`, against D7 at `:133`.

**Issue:**
D7 says `SessionIdle` "returns this reading in place of `IdleProbe`, keeping `TooShort`".
The note says "`IdleProbe` can grow a `Reason` field rather than be replaced".

**Fix:**
Pick one shape in D7 and drop or align the note.

### [LOW] A requirement lives only in an assumption

**Location:** `_lyx/discussion/decision-record.md:223`.

**Issue:**
"`fsnotify` v1.9.0 is the version to pin" is a requirement for the plan writer.
D3 and the Scope name the dependency without a version.
Under the completeness-before-leanness test, it belongs in a Decision or a Constraint.

**Fix:**
Move the pin into D3 or Constraints, and drop the assumption line.

### [NIT] Open risks restate what the Decisions already say

**Location:** `_lyx/discussion/decision-record.md:228`–`:236`.

**Issue:**
Several Open risks repeat other parts of the record:

- the first repeats D7's Bound almost word for word;
- "Backed-off orphan reap takes up to three minutes" repeats D5;
- the hook-semantics risk repeats the assumption at `:224`;
- the `/color` and `/reload-plugins` risk repeats D7's Bound at `:137`.

Their only new content is the instruction to watch the first live compaction and smoke run.

**Fix:**
Cut each bullet to the risk it adds beyond its Decision: watch the first live compaction reload, and read the transcript for the hook's ordering.
Drop the restatements.
Each carries no requirement missing from the Decisions, so the completeness test is met.

## Focus departures

None.
Every directive was reached:

- The bound question gives F4. D2, D7, D9 and D11 do state bounds.
- The D7 versus acceptance-criteria question gives F6. The criteria match D7's lists item for item, and the gap is in both.
- The premise question gives F2 and F3.
- The both-or-neither window gives F1.
- The relocation check gives F9 and F10.
