---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D2 and Acceptance criteria
    summary: D2 widens the phrases the two files and those two files, which miss the lead-in the two named in Step 5, while the acceptance criterion widens every two-files mention and so wrongly sweeps in Committing your two files
  - id: F2
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md D9; _lyx/discussion/support-log.md Rejected alternatives
    summary: D9 and the support log agree that PATTERN-daemon-wakeups does not bind a capped call, but the rule exempts only a bounded wait that ends within a minute, and the 250 s cap is over a minute
---

### [MEDIUM] D2's widening rule does not match the host section's sentences

**Location:** `_lyx/discussion/decision-record.md` D2 (second bullet) and the fourth Acceptance criterion.

**Issue:** The host template's `## What you may write` section (`contracts/stencils/loom/loom-template-discussion.md`) mentions the writer's files in three sentences:

- line 159, the lead-in: "The only files you create or modify in this session are the two named in Step 5";
- line 179: "Committing your two files is the loop owner's job, not yours.";
- line 183: "something outside those two files looks broken".

D2 has the twill paragraph widen "every mention of 'the two files' or 'those two files'".
The phrase "the two files" does not occur in the section, so the widening covers only line 183.
The lead-in at line 159, the sentence that sets the write scope, is not among the phrases named.
It still says the only files are the two, directly above a paragraph that adds a third.
The acceptance criterion says something different: it widens "every 'two files' mention in that section".
That wording takes in line 179, where widening is wrong.
The marker is untracked state under `.lyx` (D7), so it must never be described as a file the loop owner commits.
So D2 and the criterion each name a different set of sentences, and neither set is the right one.
The scope itself is consistent: D2, D7 and the criterion agree on the two discussion files plus the marker and its parent directory, and `single` stays byte-identical because the section's own sentences do not change.
D2's last bullet point admits the parent directory.

**Fix:** In D2 and the criterion, name the sentences by their role instead of by quoted phrases.
The twill paragraph widens the lead-in's write scope and the "those two files" repair rule to include the marker.
The commit sentence stays about the two discussion files only.

### [LOW] D9's reading of `PATTERN-daemon-wakeups` passes over the rule's one-minute exemption

**Location:** `_lyx/discussion/decision-record.md` D9 (the poll bullet) and `_lyx/discussion/support-log.md` Rejected alternatives ("Keeping `await-review`'s 200 ms poll").

**Issue:** The two files state the same reading.
D9 says each call "has its cap as its own deadline, so it is not an unbounded wait under that rule's 'What counts'".
The support log says "the rule itself does not bind a call with its own deadline (D9)".
`pattern/PATTERN-daemon-wakeups.md` defines an unbounded wait as a loop with no deadline of its own.
It then exempts only "a bounded wait that ends within a minute".
A call with a 250 s cap is bounded, but it does not end within a minute, so the rule leaves it neither clearly in nor clearly out.
Both files assert that it is out without addressing that sentence.
The design meets the rule either way:
the poll is at the one-second floor;
a stat spawns no process and captures no pane, so the backoff clause does not apply;
and the `fswatch` rejection gives the reason a watched parent directory may not exist yet.
The premise is the only problem.

**Fix:** In D9, drop the claim that the rule does not bind the call.
State that the poll meets the rule whether or not a capped call over a minute is in scope: the one-second floor, no backoff because a stat spawns nothing, and stat in place of `fswatch` for the stated reason.
The support log's rejected alternative then cites the floor without the "does not bind" clause.

## Focus verdicts

- **Write scope (D2):** the agreed scope holds.
  The defect is in the wording, F1.
  `single`'s byte-identity holds because the twill paragraph is an optional marker and the section's sentences stay as they are.
  D2's last bullet admits the creation of the marker's parent directory.
- **Fork-edge bound (D6): not a defect.**
  In `internal/shuttleengine/claudeengine/settings.go`, both Agent hooks are installed only when `cfg.ClaudeDenyAgentTool && !allowAgentTool`.
  The webster fork guard is keyed on `forkSubagents` alone.
  D6's bound states the false case correctly: no Agent hook, any subagent type allowed as under `single`, and the webster guard stays.
- **Relaunch rule (D4): not a defect.**
  `buildResumeCmd` in `internal/shuttleengine/claudeengine/command.go` carries no prompt pointer, which matches "types no prompt".
  The rule covers each case.
  A writer that died before step 2 resumes at step 3 and records that no forks were spawned.
  One that died during step 3 resumes at step 3 and records every fork as "no report".
  One that died after the marker resumes at its first unfinished step, keeps the findings it already holds and records the unreturned forks as "no report".
  Forks are never re-spawned, and the next turn comes from whatever next reaches the session, bounded by the run timeout.
  One reading that is less than explicit: a writer that died during step 5 has no live forks to wait for, so step 5 is moot.
  "has lost every fork it spawned" already implies that, so it is not filed.
- **Poll basis (D9 and the support log):** the two files agree with each other.
  Their shared reading of the rule is F2.

The D4 premise that a gated waiting turn end with a fork outstanding is never an arrival matches `internal/shuttleengine/doc.go`, which says such a turn end "is never an arrival" and waits "until the task's completion starts the next turn".
