---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:46-50
    summary: D1 bound says the rewrite adds no new way to corrupt a Stop or ask line, but the single printf writes in 4096-byte chunks where today's cat writes the payload in one write, and four more hooks now append to the same file
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:81
    summary: A session end as newest signal reads dead whether or not the process is found, but Claude Code fires SessionEnd with reason clear or resume in a live process, so the orch clear cycle reads dead
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:69
    summary: The orch watcher has no run record and D3 names no source for the session id the registry lookup needs, and the orch session id changes at every clear
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:110
    summary: Interrupt and transcript-marked api-error entries are listed as resting only on signals and the same for every reader, but both rest on a transcript read at reading time that sees only the newest entry
  - id: F5
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:119-120
    summary: D4 treats a dead pid in the registry as the dead signal, but Claude Code appears to remove the registry entry at exit, so a gone process mostly reads unknown liveness-unproven and the premise is not in Open risks
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:72
    summary: The wait loop folds only new signals per poll, yet idle-done must appear at a later poll with no new signal, and D3 does not say what the loop retains to re-judge the newest turn end
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:141
    summary: D6 and D3 call the newest signal-derived entry the bound on a call-time state, but the entry just before a call-time state is usually idle-done or no-output, which D3 excludes from signal-only entries
  - id: F8
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:49
    summary: Acceptance names a payload larger than one buffer without a size or test tier, and says the same event although the event's Raw bytes carry the stamp
---

### [BLOCKING] D1 bound on line corruption is false for the rewritten Stop and ask hooks

**Location:** `_lyx/discussion/decision-record.md:46-50` (D1 rationale and Bound); acceptance criterion at line 235.

**Issue:**
D1 says one `printf` per line "is not one `write`", which is honest, and it claims no atomicity.
It then says interleaving can happen "as today's `cat >> … && printf '\n' >> …` pair can" and that "the rewrite adds no new way to corrupt a line and closes none".
That last claim does not hold, and the Bound is the one place D1 states what limits the only change that touches the wait loop's input.

I measured both forms under `strace -e write` with a 20000-byte payload:

- Today's pair: `cat >>` issues one `write` of 20000 bytes, then `printf '\n'` issues a second `write` of 1 byte. A concurrent appender can only land between those two writes.
- A bash `printf '%s\n' "$in" >>`: five `write` calls of 4096, 4096, 4096, 4096 and 3617 bytes.

So a Stop payload between roughly 4 KiB and the `cat` read size, which a long `last_assistant_message` easily reaches, goes from one `write` to several, and each gap between chunks is a new interleave point.
The gaps also matter more than before because the task adds four more appenders to the same file (`UserPromptSubmit`, `StopFailure`, `Notification`, `SessionEnd`), where only Stop and the ask record wrote before.
`ParseEvents` skips a corrupted line and the wait loop classifies by the last parsed event, so a skipped Stop leaves the run waiting to its Timeout (D1 says so itself).
That is a behavior change in a task whose premise is that shadow mode changes none, caused by a choice (one `printf`) whose benefit D1 never states.

The escapes are fine, and I settled that part of the directive: a `%s` argument is not interpreted for `%`, `\`, `$` or a literal backslash-n pair, and the existing webster fork guard already uses the `in=$(cat)` then `printf '%s' "$in"` form.
`$(cat)` drops trailing newlines and NUL bytes, which a compact JSON payload does not carry.

**Fix:**
Make the Bound true.
Either state that the rewrite splits a Stop or ask line into buffer-sized writes, that four more hooks append concurrently, and carry that as an accepted Open risk with its consequence (a lost Stop line holds the run to Timeout);
or require an append form whose bytes for one line leave in one write at least up to the size `cat` handles today, and say which form.
Add the chosen behavior to the acceptance criteria.

### [MEDIUM] A session end reads `dead` even though Claude Code fires SessionEnd in a live process

**Location:** `_lyx/discussion/decision-record.md:81` (the `dead` state) and `:93` (precedence rule 1).

**Issue:**
D3 makes a session end the newest signal `dead` "whether or not the process is still found", with the session-end reason as the cause.
The 2.1.294 binary's SessionEnd reason set is `clear`, `resume`, `logout`, `prompt_input_exit` and `other`.
`clear` and `resume` end a session inside a process that keeps running.
The orch's own clear cycle types `/clear` (`internal/orchengine/doc.go`, `CycleClear`), so between that `/clear` and the next `UserPromptSubmit` the orch session reads `dead` while it is alive.
D5's orch watcher logs a Warn whenever the idle probe disagrees with the state, so every clear cycle would log a false disagreement, which pollutes the data shadow mode exists to collect.
Precedence rule 1 also overrides a live process on purpose, so D4's liveness never gets to correct it.
The Decisions and acceptance criteria name one outcome for "a session end with the process gone", but none for a session end with the process alive.

**Fix:**
Decide the state for a session end whose reason is `clear` or `resume`: for example not terminal, `unknown` with that cause until the next signal, or `dead` only for the reasons that end the process.
Add the case to the reducer acceptance row and to D3's precedence list.

### [MEDIUM] The orch watcher's liveness input has no named source

**Location:** `_lyx/discussion/decision-record.md:69` (facts list) and `:119` (D4).

**Issue:**
The directive asks that every fact the reducer reads is named with its source and that the orch watcher can supply it.
D3 names the source of run mode, output list and prompt offset for the orch (it hands interactive, no output contract, offset zero).
D4 looks the process up "through Claude's session registry entry for the session id".
For a run, `run.json` holds `sessionId` (`internal/shuttleengine/rundir.go`).
The orch has no run record, and `internal/orchengine/state.go` holds no session id, only the strand.
The orch session also gets a new session id at each `/clear`, so a stored one would go stale.
Nothing in D3, D4 or D5 says where the orch watcher gets the id, so its liveness is either always `unknown` or left for the plan writer to invent.
`lyx shuttle state` and batten read run records, so they are not affected.

**Fix:**
Name the orch's session-id source in D3's facts list (or state that the orch session reads `unknown` `liveness-unproven` by design in this task) and add the choice to the acceptance criteria.

### [MEDIUM] Interrupt and api-error history entries are not the same for every reader

**Location:** `_lyx/discussion/decision-record.md:110` (also `:74-77`, `:102-104`).

**Issue:**
D3 lists `api-error`, `interrupt` and the other signal-only entries as "the same for every reader".
Two of those rest on a transcript read at reading time, not on a signal:

- `interrupt` is read from the transcript the newest turn start names, and only whether the transcript's newest main-chain entry is the marker. A reader that folds while the marker is newest sees an `idle-stalled` `interrupt` entry; a later reader, after the transcript has grown or the next prompt arrived, never sees it, because only the newest turn start is examined. The Open risk about `UserPromptSubmit` not firing for a self-started turn widens the window.
- A transcript-marked `api-error` uses the existing `TurnEndActivity` (`claudeengine/activity.go`), which reports the newest turn-end or API-error entry in the whole transcript, not the entry of a given turn end. Applied while a files-only reader folds an older Stop, it labels that Stop with the newest entry's result.

So the claim at line 110, and the reason D3 gives for needing no persisted history, do not hold for these entries, and D3 does not say whether the transcript marker applies to the newest turn end only.

**Fix:**
Move `interrupt` and transcript-marked `api-error` into the per-reader group at line 111, and state that both marker reads apply to the newest turn end or turn start only.
Add an acceptance row for an older turn end followed by a newer one, so a transcript marker never labels the older entry.

### [LOW] A gone process mostly leaves no registry entry, so `dead` `process-gone` is hard to reach

**Location:** `_lyx/discussion/decision-record.md:119-120` (D4), `:94-95` (precedence rules 2 and 3).

**Issue:**
D4 reads `dead` from a dead pid in a registry entry and `unknown` from no entry.
On this machine `~/.claude/sessions` holds 13 entries and every pid is alive, although many earlier sessions have ended since the directory was written.
That suggests Claude Code removes its entry on exit, so a vanished process normally leaves no entry and reads `unknown` `liveness-unproven`.
`dead` `process-gone` would then need a crash or kill that leaves a stale file.
The consequence is that D5's mapping `died`↔`dead` will mostly log a disagreement against `unknown`, and the precedence example at lines 94-95 describes the rare case as the common one.
I observed this on one machine and did not test a kill, so the finding is the missing statement, not a proven defect.

**Fix:**
Verify what a normal exit and a killed pane leave behind, state the result in D4, and list the unverified premise under Open risks next to the `StopFailure` one.

### [LOW] The wait loop's fold and the later-poll `idle-done` need a stated retained input

**Location:** `_lyx/discussion/decision-record.md:72` against `:105-106`.

**Issue:**
Line 72 says the wait loop "folds each poll's new signals with that poll's stat", so a turn end is "judged by the files as they were when it first read them".
Line 105 and the acceptance row at 237 require `idle-done` to appear at a later poll when the output files appear after a `no-output` or interactive-`asking` turn end, with no new signal.
A pure fold over new signals cannot do that without keeping the previous newest turn end, or re-folding the whole file with the current stat, and re-folding contradicts line 72 for older turn ends.
D3 does not say which, so the stat does apply to every turn end in one fold (as the directive asks, this is stated), but how the newest one is re-judged at a later poll is open.

**Fix:**
State that the wait loop keeps its reducer result and re-judges only the newest turn end when the stat changes, leaving earlier history entries as logged.

### [LOW] D6 and D3 disagree on what bounds a call-time state

**Location:** `_lyx/discussion/decision-record.md:141` against `:110-112`.

**Issue:**
D6 says "the history's newest signal-derived entry shows when the session's state last changed on a signal", and D3 line 112 says "the newest signal-derived entry before it bounds how long it has held".
D3 line 110 defines signal-derived entries as those resting only on signals, and puts `idle-done`, `no-output` and interactive `asking` in the per-reader group.
The entry just before a call-time `dead` or `unknown` is usually one of those (a turn end, then the process goes), so "newest signal-derived" skips back past the turn end to an earlier turn start and overstates how long the state has held.
D6 also says the verb prints the entry for a call-time state with `since` set to the call, which matches D3, so only the bounding phrase is inconsistent.

**Fix:**
Use one term in both places, for example "the newest history entry that is not a call-time state", and say which stamp it carries.

### [NIT] Acceptance wording for the hook round trip

**Location:** `_lyx/discussion/decision-record.md:49` and `:235`.

**Issue:**
"Larger than one buffer" names no size, and the buffers differ: 4096 bytes for the shell's `printf`, the pipe size, and `cat`'s read size.
"Parses to the same event" is not literally true, since `Event.Raw` carries the stamped line (and `offsetPastEvent`, `TurnEndActivity` and the usage reads decode that Raw); every other field is equal.
The criterion also runs a real shell, and the Constraints name no test tier for that (`PATTERN-test-speed`).

**Fix:**
Name a payload size past the chosen write size, say the events are equal except `Raw`, and state the tier for the shell round trip.

## Settled without a finding

- Escapes in D1: `%`, `\`, `$` and an escaped `\n` pass through `%s` unchanged; the acceptance criterion covers them.
- D1 claims no write atomicity for the hook's shell (the false statement is the Bound, F1).
- D3 states the output-file stat applies to every turn end in one reading, and that `idle-done` carries the turn end's `lyx_at` for a files-only reader, so it has one stamp; only the retained-input gap in F6 is open.
- D3 precedence gives one outcome for each of the three cases in the directive (session end, dead pid, no registry entry with outputs present), and the acceptance rows at lines 238 repeat them; F2 and F5 concern what feeds those rules.
- D1's reason that `lyx_at` is advisory (agent-writable) and its bound (never compared with a clock) do not conflict, and D3 states that `since` values may not increase.
