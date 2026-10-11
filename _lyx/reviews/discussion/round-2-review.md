---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2, Acceptance criteria
    summary: D2 adds the marker path to the host's What you may write list but leaves the host text that says the writer modifies only the two files named in Step 5, so the rendered twill prompt still states two write scopes
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6
    summary: D6's bound says the edge refuses every other subagent type, but the fork-only Agent hook is installed only while the shuttle config's claude_deny_agent_tool is true; with it false the twill writer and its forks may spawn any subagent type
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4
    summary: The relaunch rule assumes the writer died after writing the marker; a writer that died before step 2 or before step 4 is told to finalise from step 6, which skips writing the files, the self-check, the marker, or the lens review itself with no record of the skip
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/support-log.md Rejected alternatives
    summary: The rejected 200 ms poll alternative says a 250 s call is past the minute PATTERN-daemon-wakeups exempts, implying the rule binds it, while D9 says each call is not an unbounded wait under the rule's What counts
---

### [MEDIUM] The twill prompt's write scope still reads as the two discussion files only

**Location:** `_lyx/discussion/decision-record.md` D2 (second marker), Acceptance criteria ("With `twill`, the rendered prompt's ... list names the two discussion files and the ready-marker path, and no other path").

**Issue:** The host template's `## What you may write` opens with "The only files you create or modify in this session are the two named in Step 5", and later says "If a tool you are told to run fails because something outside those two files looks broken" and "Everything else in the worktree is read-only to you".
D2 places a marker only inside the bulleted list, so the twill prompt lists three paths under a sentence that says two, and the lead-in still ties the scope to Step 5, which never names the marker.
D2 also requires `single`'s rendered prompt to stay byte-identical, so the lead-in cannot be reworded unconditionally; the record does not say how the sentence is reconciled (a marker in the sentence, or a rewording both modes share that the byte-identity criterion then relaxes).
The writer must also create the marker's `.lyx` subdirectory, a write the section does not admit, where Step 5 names its own directory creation for `_lyx/discussion/`.
The acceptance criterion "and no other path" is false as written: the same section names `_lyx/config/`, `_lyx/shed/<slug>/` and `_lyx/plan/` as forbidden paths.

**Fix:** In D2, state how the lead-in sentence and the "those two files" sentence render for `twill` while `single` stays byte-identical, and that the marker's parent directory is created by the writer.
Reword the acceptance criterion to "the files it may create or modify are the two discussion files and the ready-marker path".

### [MEDIUM] D6's fork-only bound holds only under the default shuttle config

**Location:** `_lyx/discussion/decision-record.md` D6, "Bound: the edge admits fork subagents only, refusing every other subagent type".

**Issue:** In `internal/shuttleengine/claudeengine/settings.go`, the fork-only Agent entry is `installed: func(in denyInputs) bool { return in.cfg.ClaudeDenyAgentTool && !in.allowAgentTool && in.forkSubagents }`.
`claude_deny_agent_tool` is an operator key in the shuttle config (`internal/shuttleengine/template.yaml`, default `true`); with it `false`, no Agent hook is installed at all, so the twill writer and every fork may spawn any subagent type, while the webster fork guard (`installed: in.forkSubagents`) stays.
The rest of D6 checks out against the hook: it greps the payload for `"subagent_type":"fork"`, so a fork spawned from inside a fork is admitted and refused only by the stencil, as D6 says, and D6 states that nothing but the stencil bounds a fork's writes outside the two discussion files.

**Fix:** Qualify the bound in D6: the fork-only refusal holds while `claude_deny_agent_tool` is true, and with it false the edge admits every subagent type, bounded as `single` is under the same setting.

### [MEDIUM] The relaunched-writer rule covers only a death after the marker

**Location:** `_lyx/discussion/decision-record.md` D4, last paragraph.

**Issue:** "on its next turn it records each fork that had not returned as 'no report' in step 7, finalises from step 6 with the findings it already holds, and never spawns again" is correct only if the process died after step 4.
A writer that died during step 3 has no valid record and no marker; finalising from step 6 skips writing the files, the self-check and the marker.
A writer that died before step 2 completed spawned no forks, so "never spawns again" leaves the run with no lens review, and step 7 has no forks to name, so the skipped review is recorded nowhere.
The record also leaves open what gives the relaunched writer its "next turn", since reed's resume types no prompt; D7's "keeps the marker" holds either way.

**Fix:** State the rule per step reached: resume at the first unfinished step of 3 to 8, never re-spawn, and record in step 7 that forks were lost or never spawned.
Name what prompts the relaunched writer's next turn (a gate re-prompt, or none, in which case the run's existing timeout ends it).

### [LOW] The support log and D9 disagree on whether PATTERN-daemon-wakeups binds the wait

**Location:** `_lyx/discussion/support-log.md` `## Rejected alternatives`, "Keeping `await-review`'s 200 ms poll for the shared wait".

**Issue:** The rejected alternative says "A 250 s call is past the minute `PATTERN-daemon-wakeups` exempts", which reads as the rule binding each call.
D9 says the opposite: each call has its cap as its own deadline, so it is not an unbounded wait under "What counts".
The rule text in `pattern/PATTERN-daemon-wakeups.md` defines an unbounded wait as a loop with no deadline of its own and exempts a bounded wait under a minute, and says nothing of a bounded wait over a minute, so D9's reading is the one the record should state once.
D9's basis otherwise holds: a one-second stat poll meets the floor, needs no `//lyx:one-shot` annotation, and the existing poll in `internal/burlercli/await.go` uses `time.After`, which the guard's `time.NewTicker`/`time.Tick` scan does not catch, so no guard-list entry is required unless the plan picks a ticker.

**Fix:** Reword the rejected alternative's reason to D9's basis: one stat a second is the floor the rule sets for waits it governs, and costs a cache-warming waiter nothing, rather than claiming the rule binds the call.

## Focus verdicts

- Write scope: F1.
- Fork-edge attack surface: F2; the fork-from-fork admission and the write-containment statement are accurate.
- Writer lifecycle: F3.
- Poll basis: D9 and the Constraints line match the rule text; only the support log contradicts them (F4); no annotation or guard-list entry is needed for a `time.After` poll at one second.
- Marker vocabulary: not a defect. D3's markers cover every host-specific value D2, D4 and D5 use (reviewed files, marker path, rubric, lens list, disposition location, format and self-check steps, self-check command); the await command and the fork boilerplate are host-neutral.
