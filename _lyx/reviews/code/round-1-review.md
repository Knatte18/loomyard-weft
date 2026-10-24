---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/activity.go:186
    summary: A parse failure of only the pre-offset part, after the own part parsed, logs a Warn and an Info on every poll because Recover ends the episode that Fail then restarts
    fix: In newestTurnEnd, parse both parts first and call unreadableFiles.Recover once after both parse, instead of inside the loop after each part. Fail stays at the failing part. Add a row to TestReadAgentActivity whose engine fails the second ParseEvents call only, asserting the Warn is logged once across repeated reads.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/activity.go:181
    summary: A PromptOffset past the end of the file now reads the whole file as the pre-offset part, where the old code parsed nothing, and no test row pins that behaviour
    fix: Add a row to TestReadAgentActivity with events "STOP:skill\n" and a promptOffset larger than its length, asserting the transcript's last activity is reported and no API error, so the clamp's widened result is pinned.
    origin: lens:code-tests
---

### [LOW] Recover-then-Fail flap when only the pre-offset part fails to parse

**Location:** internal/shuttleengine/activity.go:186
**Issue:** The loop calls `Recover` after the own part parses, then `Fail` when the pre-offset part does not. `EpisodeSet.Recover` deletes the key's episode (`internal/logger/episode.go:70`), so the next poll's `Fail` starts a new episode and logs at Warn again, with an Info at each recovery. The old single parse could not fail on this input. Reachable only through an engine whose `ParseEvents` errors; the Claude engine never errors (`claudeengine/events.go:72`), so there is no effect today.

### [LOW] Past-end prompt offset widened without a test

**Location:** internal/shuttleengine/activity.go:181
**Issue:** `min(max(PromptOffset, 0), len)` keeps the old result for a zero and a negative offset. For an offset past the end, the own part is empty and the pre-offset part is the whole file, so a turn end is now read where the old code read none. The change can only make a run read more recently active and drop the API error, so it is harmless. No row covers it.

## Rejected

- The `ownTurnEnd` gate and the `LastActivity` merge were settled as correct: the API error is dropped for every pre-offset reading, and the reading can only raise the run's last activity.
- An offset off an event boundary cannot make a torn line fail the parse, since the Claude `ParseEvents` skips malformed lines.
- Affected set (callers in `battencli`, `loomcli`, docs): none left wrong.
- Pattern fit and comment conventions: no violation.
- Acceptance criteria 1 and 2 are pinned by the new row, criterion 3 by "turn ends before the prompt offset are not the run's own", and criterion 4 by "a run with no turn end yet is judged by its events file alone".
