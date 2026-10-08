---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: decision-record.md D3 (since stamps)
    summary: D3 leaves the since stamp undefined for unknown from no signal or a contradiction, for interactive asking from a missing output, and for idle-done reached when output files appear
  - id: F2
    severity: MEDIUM
    class: consistency
    location: decision-record.md D1 (lyx_at trust) against D3 (interrupt rule)
    summary: D1 says only since and history read lyx_at, but D3's interrupt rule orders a transcript timestamp against the turn start time, so the state itself depends on the agent-writable stamp and no tie or skew rule is given
  - id: F3
    severity: MEDIUM
    class: design
    location: decision-record.md D1 (first-seen fallback) against D3 (same history for every reader)
    summary: A line without lyx_at reads at the time the reader first saw it, but a files-only reader keeps no memory, so since is undefined there and the history is not the same for every reader
  - id: F4
    severity: MEDIUM
    class: design
    location: decision-record.md D3 and D6, support-log.md Rejected alternatives (persisted state file)
    summary: The since of dead and unknown from a call-time fact is the reading time, so lyx shuttle state restarts it on every call, and the stated reason for rejecting a persisted file does not cover these states
  - id: F5
    severity: MEDIUM
    class: consistency
    location: decision-record.md D8, D5 and the matching acceptance criteria
    summary: D8 states Done-at-once and keeps-waiting without saying they hold only for a gated run, because the loop already returns Done for any run whose output files exist, forks included, before it consults the shell expiry
  - id: F6
    severity: MEDIUM
    class: design
    location: decision-record.md D1 (Bound) and Stop hook rewrite
    summary: D1 rewrites the Stop and ask hook that the wait loop reads and makes a failed append silent, so a dropped turn end now changes behavior in a shadow-mode task with no stated fallback
  - id: F7
    severity: MEDIUM
    class: decision
    location: decision-record.md D11 and Open risks
    summary: D11 never names the payload field the command-position match reads, while Open risks says the regex runs over the whole JSON payload, so the false-deny class stated in D11 is incomplete
  - id: F8
    severity: LOW
    class: design
    location: decision-record.md D11 (Bound, switch timing)
    summary: D11 states that a mid-run edit of shuttle.yaml changes no running session but not that a session spawned afterwards in the same worktree reads the edited switch, nor which code reads it
  - id: F9
    severity: LOW
    class: consistency
    location: decision-record.md D1 (Bound) against D10 and D11
    summary: D1 says every hook command prints nothing, but the deny hooks D10 and D11 keep and add print the deny JSON by design
  - id: F10
    severity: LOW
    class: decision
    location: decision-record.md D3 (facts, interrupt marker)
    summary: The interrupt marker has no named transcript path source or marker shape, and the existing ActivityReader reads only the API-error marker from a Stop payload's transcript_path
---

### [MEDIUM] since is undefined for several states

**Location:** `decision-record.md` D3, the paragraph on state changes.
**Issue:** D3 names a stamp for signal-derived states, `no-output`, a transcript-marked `api-error`, `interrupt`, and `dead` and `unknown` "from a call-time fact".
Four cases have none.
`unknown` with no signal past the prompt offset is not a call-time fact and has no signal to stamp it.
`unknown` from the idle-notice contradiction is signal-derived, but the text does not say whether it takes the idle notice's `lyx_at` or the turn start's.
`asking` from an interactive turn end with an output file missing is stamped only for the autonomous `no-output` case.
`idle-done` reached because the output files appeared after a `no-output` turn end rests on a call-time fact, and the text names neither the turn end's time nor the time of the reading.
**Fix:** In D3, state one stamp per state and cause, including these four.

### [MEDIUM] The state depends on the agent-writable stamp

**Location:** `decision-record.md` D1 (last sentence) and D3 (`busy` and `interrupt`).
**Issue:** D1 bounds the trust: "only the reducer reads it, for `since` and the change history, and no decision, notice or return depends on it".
D3 makes `idle-stalled` `interrupt` depend on an interrupt marker "after the newest turn start", with the entry's own transcript timestamp.
That comparison is between a transcript timestamp and the turn start's `lyx_at` (or its first-seen time).
So the state, and not only `since`, depends on the stamp.
A forged or skewed `lyx_at` flips `busy` to `idle-stalled` or hides an interrupt, and the stamp has second resolution while a transcript timestamp does not, so a same-second tie has no rule.
The state is shadow, so this bounds what a later consumer inherits rather than what this task breaks; D1 should say it.
**Fix:** In D1 and D3, state that the reducer reads `lyx_at` for ordering as well, define the tie and skew rule for the interrupt comparison (for example by file order rather than time), and restate the bound accordingly.

### [MEDIUM] First-seen time has no holder

**Location:** `decision-record.md` D1 ("A line without `lyx_at` ... reads with the time the reader first saw it") and D3 ("The signal-derived part of the history is the same for every reader").
**Issue:** `lyx shuttle state`, batten and the orch watcher read files only and persist nothing.
A reader that sees a legacy or malformed line for the first time on every call has no first-seen time to report, so it can only use the time of the call.
Then `since` for those lines moves on each call, and the two readers' histories differ, which D3 says they do not.
**Fix:** State what a reader without memory uses (for example the events file's mtime or the reading time) and limit D3's "same for every reader" claim to lines that carry `lyx_at`.

### [MEDIUM] since moves on every call for call-time states

**Location:** `decision-record.md` D3 ("the time of the reading for `dead` and `unknown` from a call-time fact"), D6; `support-log.md` Rejected alternatives, "The Wait loop as the sole writer of a persisted `state.yaml`/history file".
**Issue:** The rejection reason is that the orch has no Wait and that deriving from the events file gives every reader the same signal-derived history.
For the signal-derived states that holds.
It does not hold for `dead` with cause `process-gone` and for `unknown` from liveness or an unreadable file, which D3 itself calls per reader.
A session that was killed writes no `SessionEnd`, so `lyx shuttle state` prints `dead` with `since` equal to the time of each call, and the operator cannot read how long it has been dead, which is what the verb is for.
The wait loop's own log is unaffected because it logs a change once.
**Fix:** In D3 and D6, state that `since` for a call-time state is the reading time and restarts on every call, or give a stable source (for example the newest signal's time as "no later than"), and narrow the support-log reason to the signal-derived states.

### [MEDIUM] D8's Done-at-once and keeps-waiting ignore the gate

**Location:** `decision-record.md` D8 (third and fourth sentences), D5 (third pairing), Acceptance criteria (the "finishes Done at once" bullet).
**Issue:** `Run.pollEventsTick` in `internal/shuttleengine/wait.go` returns `OutcomeDone` as soon as `allOutputFilesExist` holds, whatever the last event's kind and whatever is outstanding, before it reaches `expiredTurnEnd`.
Only a gated run (`len(run.gate) > 0`, webster's Master) defers a waiting turn end to `expiredTurnEnd`, whose shell expiry is what D8 changes.
So for a run without a gate, outputs present already means Done at once with a fork or an awaited shell outstanding, and no `ExpiredShells`.
The Decision and its acceptance bullet say "with an outstanding fork or awaited-prefix shell it keeps waiting" and describe Done-at-once for payload-reported shells without naming the gated run as the condition.
A plan writer who implements it as written either changes the ungated runs' behavior or writes a test that fails against the code.
D5's "expected pairings" have the same gap: Done with an outstanding fork reads `busy` cause `background` against `done` and is not listed, so it would log a spurious disagreement.
**Fix:** State in D8 that the rule concerns a gated run, say what an ungated run does today (unchanged), and either list Done with an outstanding fork among D5's expected pairings or define the pairing by "wait loop `done` with `busy` cause `background`".

### [MEDIUM] Rewriting the Stop hook can drop the turn end silently

**Location:** `decision-record.md` D1 (the `lyx_at` insertion and "Bound").
**Issue:** The Stop hook today is `cat >> events && printf '\n' >> events`, and the wait loop's classification reads that file only.
D1 changes this hook (and the ask record) to a single write with a field inserted as the first field, and requires every hook command to exit 0 "whether or not the append succeeds".
A payload the insertion cannot handle, or a failed append, now loses the turn end without any error, where today the failing command is visible as a hook error.
That is a changed guard in a task whose premise is that shadow mode changes no behavior, and the Decision states what exit 0 hides from the user but not what it lets through for the wait loop.
**Fix:** In D1, state that when stamping the line fails the hook appends the unstamped payload as today, so the Stop and ask lines are never dropped, and keep the silent-failure bound for the new hooks only.

### [MEDIUM] D11 does not name the field the command-position match reads

**Location:** `decision-record.md` D11 ("Decision" and the false-deny sentence) and Open risks (last bullet).
**Issue:** Open risks says the matcher is "a regex over the JSON payload", and the existing webster fork guard in `settings.go` greps the whole payload.
A `PreToolUse` Bash payload carries `description`, `cwd` and other fields besides `tool_input.command`.
A whole-payload match at "the start" (the start of a JSON string) or after the escaped `\n` can match `description` or another field, so the false-deny class D11 lists (quoted argument, heredoc body) is incomplete.
"After a newline" also means the two-character JSON escape, which the Decision does not say.
**Fix:** In D11, name the payload field the match is anchored to, or add the fields it can falsely match to the stated false-deny class.

### [LOW] Timing of the python switch for later sessions

**Location:** `decision-record.md` D11, last sentence.
**Issue:** The text covers a running session and says the switch "is read when a session's settings are written at spawn".
It does not say that a session spawned later in the same worktree reads the edited `shuttle.yaml`, which an agent can edit, nor which code reads it.
Another way an agent turns the guardrail off is therefore stated for one case only.
**Fix:** Add that a later spawn, including a resume that rewrites settings if it does, reads the current value, and name the reader (the settings writer in `claudeengine`, through `LoadConfig`).

### [LOW] "Every hook command prints nothing" is too broad

**Location:** `decision-record.md` D1, "Bound".
**Issue:** The deny hooks that D10 keeps and D11 adds print the deny JSON on standard output by design.
**Fix:** Scope the sentence to the recording hooks (`UserPromptSubmit`, `StopFailure`, `Notification`, `SessionEnd`, Stop and the ask record).

### [LOW] Interrupt marker has no source

**Location:** `decision-record.md` D3, the fact "The transcript's interrupt marker after the newest turn start".
**Issue:** D3 names `claudeengine` as the source and the existing `ActivityReader` only for the API-error marker.
`TurnEndActivity` takes a Stop event and reads its `transcript_path`, and reports the API-error marker only.
For the interrupt marker the Decision names neither the payload that supplies the transcript path (a `UserPromptSubmit` line, since the interrupt follows the turn start) nor the marker's shape, and it does not say this is a new capability beside `ActivityReader`.
**Fix:** In D3 or D2, state that the interrupt read is new, which signal supplies the path, and what the reader returns.

## Checked and not a finding

- D8 for a payload-reported shell that never finishes: the Decision states the outcome at Timeout (`OutcomeTimeout`), that only the Info log line reaches anyone, and that the quiet notice stays suppressed while only non-awaited shells are outstanding (matches `syncShellWait`).
- D8 with a fork and a shell outstanding together: stated (no mark, quiet notice not suppressed, run keeps waiting to Timeout).
- D3, D5 and D8 on authority: D5 states that the wait loop wins; the gap is F5's unlisted pairing, not the authority rule.
- D11 states what it lets through, what it falsely denies and that it is a guardrail; F7 and F8 concern completeness only.
- "Prompt offset" in D3 is an existing term of the activity reader (`doc.go`), not an undefined one.
- No relocation or exclusion finding: no content in `decision-record.md` carries only deliberation that belongs in `support-log.md`.

## Prior rounds (read after this review was saved)

No round-1 fix regressed: the D1 bound, D11's guardrail and let-through classes, D8's Timeout outcome and log line, D5's three pairings and the corrected rejection reason are all present in the target.
F1, F3 and F4 above refine round-1 fixes (stamps for the remaining causes, the per-reader history, the gated-run condition) rather than reopening a disputed or deferred item; round 1 disputed and deferred none.

## Focus departures

None.
