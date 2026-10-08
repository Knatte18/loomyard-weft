---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:89-94
    summary: D3 gives idle-done two since sources for one input, and a memoryless reader cannot tell output files that appeared after the turn end from files present at it, so the history is not the same for every reader
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:40-47
    summary: D1 rests its one-write-per-line rationale on a shell printf that splits large payloads into several writes, and its bound does not cover a Stop or ask line that is stamped but corrupted, which ParseEvents silently drops
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:73-81
    summary: D3 states no precedence among process liveness, session end, the newest signal and the interrupt marker, so a finished or killed run with no registry entry reads both idle-done and unknown
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:48
    summary: D1 calls lyx_at advisory because the agent can write the directory, but every events line and transcript_path is equally agent-writable, and the text sets no rule on non-monotonic since values in the history
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:122
    summary: D6 says the newest signal-derived history entry shows when the session last reported anything, but the history records changes only, so repeated identical signals leave it older than the last report
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:81
    summary: D3 defines unknown by no signal yet past the prompt offset, but the offset is not among the facts the caller hands the reducer, and the orch watcher has no run record to take it from
---

### [MEDIUM] idle-done has two since sources and the same-history claim fails for output-file facts

**Location:** `_lyx/discussion/decision-record.md` D3, lines 86, 89 and 92-94.
**Issue:**
The `since` list gives `idle-done` the turn end's `lyx_at` ("a turn end with every output present") and, in the call-time bullet, the time of the reading ("`idle-done` reached because output files appeared after a `no-output` or interactive-`asking` turn end").
A files-only reader (`lyx shuttle state`, batten, the orch watcher) keeps nothing between calls, so it sees one turn end and one stat result.
It cannot tell "the files appeared after the turn end" from "the files were present at the turn end", and the text names no signal that separates the two.
Both bullets match the same input, so the stamp is not exactly one.
The same dependency reaches past the newest turn end.
The output-file list is a fact stat'ed once at reading time (line 64), and the fold applies it to the signals in file order.
Whether an earlier turn end that was `no-output` when it happened reads `idle-done` today, because the files exist now, is left open.
If it does, the history entry for that turn end is rewritten by a later reader, and the claim "the signal-derived part of the history is the same for every reader" (line 92) is false for every entry that rests on an output-file check.
If the fold applies the stat to the newest turn end only, the text must say so.
**Fix:**
Decide where the output-file fact is applied (the newest turn end only, or every turn end) and say so in D3.
Then name which readers can produce the "appeared after" stamp (the wait loop, which sees the change) and give a files-only reader one stamp for `idle-done` (the turn end's `lyx_at`).
Add the output-file-dependent entries to the list of entries that are per reader.

### [MEDIUM] D1's single-write rationale is false for large payloads, and the bound stops at the unstamped fallback

**Location:** `_lyx/discussion/decision-record.md` D1, lines 40-47.
**Issue:**
The Decision writes every hook line "in one write", and the Rationale says "one write per line keeps concurrent hooks from interleaving".
The hook command is a POSIX shell with `printf`.
I ran `bash -c 'in=$(cat payload); printf "%s\n" "{\"lyx_at\":\"x\",$in}" >> out'` under `strace -e write` on a 20000-byte payload: it issued five `write(1, …)` calls of at most 4096 bytes each, not one.
A Stop payload carries `last_assistant_message` and can exceed that size, so the property the Rationale relies on does not hold for exactly the large Stop lines.
The exposure is not new (today's `cat >> … && printf '\n' >> …` is two writes), but the Decision presents the rewrite as closing it.
The Bound covers two failure modes: the stamp cannot be inserted (line appended unchanged) and the append fails (exit status as today).
It does not cover a line that is stamped and appended but corrupt, whether by interleaving or by a stamp inserted into a payload that is not a plain non-empty object.
`ParseEvents` skips malformed lines without a trace (`internal/shuttleengine/claudeengine/events.go:33`), and the wait loop classifies by the last parsed event.
A corrupt Stop or ask line is therefore a lost turn end, and the run waits until its Timeout.
This task's premise is that shadow mode changes no behavior, and the Stop and ask rewrite is the one change in it that touches the wait loop's own input, yet the Decision states no bound for it beyond "keep parsing exactly as before" in the acceptance criteria.
**Fix:**
Drop the claim that one `printf` is one write, or name the mechanism that makes it so.
State in D1 that the rewrite of the Stop and ask commands is the one shadow-mode change that touches the wait loop's input, and bound it: the stamped line must parse to the same event as the unstamped one, including for a payload larger than one buffer and one containing `%`, `\`, `$` and an embedded `\n` escape.
Add that case to the acceptance criteria.

### [MEDIUM] No precedence among liveness, session end, the newest signal and the interrupt marker

**Location:** `_lyx/discussion/decision-record.md` D3 lines 73-81 and D4 lines 100-102.
**Issue:**
Each state is defined by its own condition, and nothing orders the conditions.
Cases the text leaves open:
(a) A run's process was killed after its last turn end and its registry entry is gone.
The newest signal is a turn end with every output present (`idle-done`), liveness is unproven (`unknown`, D4: "no registry entry … is `unknown`"), and no session end was recorded because the kill gave the hook no chance.
Both rules match, and the acceptance criteria list both outcomes without saying which wins.
(b) The process is gone and a session end is the newest signal: the cause is `process-gone` or the session-end reason.
(c) A turn start is the newest signal, the process is alive and the interrupt marker follows it: `busy` or `idle-stalled`.
The text answers this one (line 75).
A plan writer must pick (a) and (b) with no guidance, and the choice decides what the wait loop's disagreement log reports for every finished run.
**Fix:**
Add one ordering sentence to D3 (for example: a provable dead process, then a session end, then liveness-unproven, then the signal-derived states), or state that a finished run reads `unknown` by design.
Settle the cause for (b).

### [LOW] The advisory-stamp reason covers the whole events file, not only lyx_at

**Location:** `_lyx/discussion/decision-record.md` D1, line 48 (with D3 lines 64-69).
**Issue:**
The Decision calls `lyx_at` advisory because it is "written by a shell in a directory the agent can write", and bounds it: the reducer reads it only for `since` and the history, orders by file order, and no state depends on it.
That bound holds: D3 decides "after the newest turn start" by transcript order alone, and the acceptance criteria pin it (an interrupt marker reads `idle-stalled` whatever the next turn start's `lyx_at` is).
The reducer therefore does not compare a transcript timestamp with `lyx_at`, and no tie or skew rule is needed for the state.
The stated reason is wider than the bound, though.
Every line in the events file, `transcript_path` included, is as agent-writable as the stamp, so the state as a whole is as forgeable as `lyx_at`; a reader following the Decision's own logic would conclude the state is advisory too.
Separately, `since` values in the history can run backwards, because the interrupt entry carries the transcript's own timestamp and the others carry hook-side stamps from a different clock read with second resolution.
The text orders the history by file order but does not say that `since` values are not monotonic.
**Fix:**
Say in D1 that the whole state is advisory in shadow mode for the same reason, with no consumer yet acting on it, and say that the history is ordered by signal order and its `since` values are not guaranteed to increase.

### [LOW] The history bounds the last change, not the last report

**Location:** `_lyx/discussion/decision-record.md` D6, line 122 (and D3 lines 93-94).
**Issue:**
D6 says "the history's newest signal-derived entry shows when the session last reported anything".
D3 records an entry "only when the state or its cause changes", and the acceptance criteria require none "for a repeated identical signal".
A session that reports a waiting turn end (`busy`, cause `background`) several times leaves the entry of the first one as the newest, so the entry shows the last change and the reported time can be older than a later report.
For D3's use ("bounds how long it has held") this is still an upper bound and is correct.
**Fix:**
Reword D6 to "when the session last changed state" or drop the clause.

### [LOW] The prompt offset is used but never handed to the reducer

**Location:** `_lyx/discussion/decision-record.md` D3, line 81 (facts list at lines 61-69).
**Issue:**
`unknown` includes "no signal yet past the prompt offset".
The offset is a run-record value (`PromptOffset` in the shuttle run state), but the D3 facts list names only run mode, output files, existence, liveness and the two transcript reads.
The orch watcher "has no run record" (line 63), so it has no offset to hand over.
**Fix:**
Add the prompt offset to the facts list with its source, and say what the orch watcher hands (offset zero).

## Focus verdicts

1. **D3 `since` stamps.** Every state and cause names one stamp except `idle-done` reached after output files appear, which has two sources for one input (F1); `unknown` from no signal or an unreadable file, `unknown` from the idle-notice contradiction and interactive `asking` from a missing output file each name exactly one.
2. **D1 against D3 on a line without `lyx_at`.** The text does not say the reader uses its first-seen time; D1 and D3 both say the state carries an empty `since`, the same for every reader, so there is no conflict there.
   D3's same-history claim holds for lines with and without `lyx_at`, and fails only for entries resting on output-file checks (F1).
3. **Call-time `since` across repeated calls.** D3 and D6 state that a files-only reader restamps with each call and the wait loop logs once; the support log's Rejected alternatives entry covers those states by deferring a durable `since` to the first consumer.
   That is a stated reason, not a gap; the D6 wording of the bound is F5.
4. **Stop and ask hook rewrite.** Stamp failure appends unchanged and append failure exits 1 as today, both stated; the missing piece is a bound for a stamped but corrupt line and the false single-write premise (F2).
5. **State dependence on `lyx_at`.** The state does not depend on it, and D1, D3 and the acceptance criteria say so; no tie or skew rule is needed.
   The remaining gap is the width of the stated reason and the non-monotonic `since` (F4).
