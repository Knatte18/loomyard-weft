---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4 Bound, Auto-mode assumptions, Open risks
    summary: D4 says the held-run deadline is hours for loom and webster runs, but several callers set shorter deadlines and several autonomous callers wire no notifier, and Open risks omits the no-notifier case
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5
    summary: D5 routes agent-authored text into the orch session but leaves "data not instruction" as an unowned claim and bounds only newlines, not other control characters
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D5, D6
    summary: A legacy asking record with no AskingOffset replays from the prompt offset, and D6 does not say whether every earlier turn end without outputs in that backlog is notified or only the last
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2, Open risks
    summary: Open risks states the transcript-fallback condition as the key being dropped, while D2 and the acceptance criteria also fall back on a non-list value
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D1 Rationale, Notes for the plan writer
    summary: D1 Rationale cites the two transcripts by session and task id, which the Notes for the plan writer already cite by path and line, so one of the two is a duplicate
---

### [MEDIUM] The stated bound on a held run does not match the autonomous callers' deadlines

**Location:** `_lyx/discussion/decision-record.md` D4 Bound, Auto-mode assumptions (last item), Open risks (second item).

**Issue:**
D4 says the deadline "is hours for loom and webster runs, not minutes".
I checked each caller's `Spec.Timeout` in the code.
The Discussion, Review and Plan runs read `discussion_timeout_min`, `review_timeout_min` and `plan_timeout_min` from `internal/loomengine/template.yaml`, and webster's Master reads `master_timeout_min` from `internal/websterengine/template.yaml`; all of these are hours.
The same sentence also covers "loom's and webster's `*_timeout_min` keys", and those include `friction_timeout_min` (`internal/loomcli/run.go`) and webster's `recovery_timeout_min` (`internal/websterengine/recoverbatch.go`), which are tens of minutes.
Landing's conflict and describe runs (`conflict_timeout_min`, `describe_timeout_min` in `internal/landingshed/template.yaml`) and a standalone `lyx shuttle run` (`run_timeout_min` fallback in `internal/shuttleengine/template.yaml`) are also minutes.
The other half of the focus question holds: `Attach` computes `now + Timeout` per attach (`internal/shuttleengine/attach.go`), as D4 says.
D4 and D5 do state what a no-notifier autonomous run does on an asking stop: it holds silently until its deadline, logging the hold at Info.
Two gaps remain around that statement.
Open risks and the Auto-mode assumptions bound only the "orch misses the notice" case and assume the parent of every autonomous agent is the prime's orch, so they never name the callers that wire no notifier and used to stop at once on an ask.
D4 then states a bound ("hours") that a reader takes as the worst case for the no-notifier runs too.

**Fix:**
In D4's Bound, replace "so it is hours for loom and webster runs, not minutes" with the derivation: the bound is the spawning caller's own timeout key, from tens of minutes to hours depending on the caller, and `run_timeout_min` where the caller sets none.
Do not pin the values.
Add an Open risks item: an autonomous run whose spawner wires no notifier (D5) holds silently until its deadline, where before it stopped at the ask.

### [MEDIUM] The hold notice quotes agent text into the orch with no owner for "data, not instruction"

**Location:** `_lyx/discussion/decision-record.md` D5.

**Issue:**
D5 adds a routing edge: the start of a subordinate agent's last message is typed into the orch session, which acts with higher authority.
The decision answers the focus question on length (200 characters), fixed prefix (stated, text left to the plan) and the data-not-instruction statement.
The data statement is written as a fact about the orch ("the orch reads it as a report of what the agent said, never as an instruction to the orch"), not as a property of the notice or a duty on a named owner.
Nothing in D5 or D8 says the notice text itself marks the quote as quoted data, and D8's orch stencil change covers only the message tail.
`QueueNotice` refuses only a newline and a leading `/` (`internal/orchengine/notice.go`), and the watcher types the line with `Session.Send`.
D5 collapses newlines but names no handling for other control characters (an escape sequence or a Unicode line separator) in agent text, and does not say whether 200 is bytes or characters, which decides whether a multibyte character can be cut in half.
The edge's bound is therefore only partly stated.

**Fix:**
In D5, state that the fixed prefix labels the quoted text as the agent's words and not a command, and that the quote is delimited; name the owner of the statement (the notice text, plus a line in the orch stencil's Messaging section if the plan needs it).
State that control characters are replaced along with newlines and that the cut is on characters.
Add the matching assertion to the notifier acceptance criterion.

### [MEDIUM] A legacy asking record without an offset may notify more than one old turn end

**Location:** `_lyx/discussion/decision-record.md` D6 (third paragraph), D5 (last paragraph).

**Issue:**
D6 attaches a legacy record with no `AskingOffset` "from the prompt offset, so its last turn end is read as a held turn end and notified once".
Replaying from the prompt offset reads every turn end in the run's events, not only the last (`reconstructAndWait` in `internal/shuttleengine/attach.go` does exactly that today).
D5 notifies once per held turn end and suppresses only turn ends at or before the persisted notified offset, and a legacy record has none.
An agent that ended several turns without outputs after the old halt (the operator answered it, then it ended again) would notify each one in a burst, contradicting "notified once".
The record leaves open whether the replay coalesces a backlog to its last held turn end or notifies each.
Focus part 4 is otherwise settled: the legacy no-offset ask is notified on attach, and the notified offset is persisted before the notifier call.

**Fix:**
Add to D6 or D5 the rule the author picks.
Recommended: a replayed backlog notifies only its last held turn end, since earlier ones are history the parent cannot act on.
Add that case to the legacy-record acceptance criterion.

### [LOW] Open risks and D2 state the transcript-fallback condition differently

**Location:** `_lyx/discussion/decision-record.md` D2 (second paragraph), Open risks (first item), Acceptance criteria (second item).

**Issue:**
D2 and the acceptance criteria fall back to the transcript for a payload without the key or whose value is not a list.
Open risks says the fallback applies if a release "could drop `background_tasks`", which names only the absent key, and moves on to a changed entry shape or status under the same key.
A key kept with a non-list value is a third case that Open risks never mentions.
D2 does say what the two other focus cases count as: a non-list value falls back, and an entry with a status other than `running` counts as not outstanding with no transcript correction.
The only live data I found in the hub's events files carries `running` alone, so the record's "no such case is known" holds.

**Fix:**
Make Open risks state the condition as D2 does: key absent or value not a list falls back to the transcript.

### [NIT] D1 Rationale and the plan-writer note cite the same transcripts twice

**Location:** `_lyx/discussion/decision-record.md` D1 Rationale and Notes for the plan writer (last item).

**Issue:**
D1's Rationale lists the session ids and task ids of the two verifying transcripts.
The last Notes item gives the same sessions by full path and line number, which D10's corpus build needs.
The ids in the Rationale are evidence trail, not a requirement the plan writer needs beyond the Notes.
This is a relocation finding: the completeness-before-leanness test passes, since the Notes already carry the paths the plan writer needs.

**Fix:**
Cut the session and task ids from D1's Rationale, leaving "verified against both transcripts on disk"; the support log's Interview item 1 may carry the ids if wanted.

## Focus departures

None.
