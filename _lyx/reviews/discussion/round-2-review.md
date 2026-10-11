---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D5 and Constraints (PATTERN-daemon-wakeups)
    summary: D5 places awaitStartup outside PATTERN-daemon-wakeups by a reading the background's own bounded-wait sentence contradicts, and its at-the-floor argument does not answer the backoff rule a pane-capturing wake falls under
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D4, cause-present sentence
    summary: D4 says a non-busy prompt delivery failure reports no screen and gets no tail on purpose, but the not-landed and in-draft send failures already end with the pane's last lines through withPaneTail
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D4, trigger sentence
    summary: D4 says the trigger is never-ready and not a nil cause, yet the reed-status retry exits are never-ready and get no tail; the trigger is the conjunction of both
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D6, played-dismissal rule
    summary: D6 counts a play that failed partway as no play, so keys that reached the pane start no gap and the next one-second probe may dismiss again, the stray-keypress hazard D6 exists to close, with no stated bound
---

### [MEDIUM] D5's reason for exempting `awaitStartup` disagrees with the daemon-wakeups background

**Location:** `_lyx/discussion/decision-record.md` D5's last rationale sentence and the `PATTERN-daemon-wakeups` line under Constraints.

**Issue:** `pattern/PATTERN-daemon-wakeups.md` defines an unbounded wait as one that "has no deadline of its own", then treats bounded waits in one sentence: "A bounded wait that ends within a minute is outside the rule."
That sentence is the background's only exemption for a bounded wait, and it is scoped by duration.
`awaitStartup`'s window is `startup_timeout_s`, 90 s in `internal/shuttleengine/template.yaml`, so it does not meet it; the decision record concedes as much ("even though its window ... can exceed the minute the background names") and then exempts the loop anyway on the definition sentence alone.
The background does not settle a bounded wait over a minute, and its one sentence on the subject points the other way, so the record adopts a contested reading of a listed constraint without recording it in the background.
The fallback "one probe a second is at the rule's floor anyway" answers the floor rule only.
The rule that binds this loop is the backoff rule: "A wake that spawns a process or captures a pane doubles its wait from a base to a named ceiling while nothing changes."
After D5 every pre-ready probe runs `reed.Status` and `CapturePane` (`checkLivenessTick`, `internal/shuttleengine/wait.go`) once a second for up to 90 s, which is the shape that rule targets, and D5 decides "no backoff".
`internal/shuttleengine/wait.go` is on the guard's declared list in `cmd/lyx/daemonwakeups_test.go`, so the scan will not catch a computed interval; review is the only check.
Round 1's F3 offered the definition reading as one acceptable fix, and the record now states it openly, so this is not a clear breach of the listed constraint and stays MEDIUM.
What remains open is that the reading lives only in the decision record: the background still says the opposite to its next reader, and the floor argument answers the wrong rule.

**Fix:** Settle the reading in the background rather than in the record, choosing one:
(a) give `awaitStartup` the backoff the rule names, base `pollInterval(cfg)`, ceiling the dismissal gap, reset when the capture changes; a provider ready within a second is still seen on the first or second probe, so D5's goal survives;
or (b) amend `pattern/PATTERN-daemon-wakeups.md` in the same commit so a wait with its own deadline and a count cap is outside the rule whatever its length, and state why per-second pane captures for up to `startup_timeout_s` are acceptable.
Either way, drop "at the rule's floor anyway" from D5 and from the Constraints line, and make the Constraints line cite whichever reading lands.

### [MEDIUM] D4's cause-present sentence misdescribes which causes already carry a pane tail

**Location:** `_lyx/discussion/decision-record.md` D4, the sentences after "When a cause is present, nothing is added."

**Issue:** D4 says a busy-send cause carries the pane's last lines and that "a non-busy prompt delivery" reports "a mechanism or delivery failure rather than a screen, and get[s] no tail on purpose".
The send path contradicts that.
`sendVerified` in `internal/shuttleengine/run.go` ends its `ErrSubmissionNotLanded` failures with `withPaneTail` (the closed-window and never-appeared returns), `failUnlanded` in `internal/shuttleengine/send.go` does the same for a box path that did not land, `pasteVerified` does it for a closed window, and the in-draft and foreign-text box readings return `withPaneTail` too.
Only a raw `playInputs` or `PasteSubmit` error reaches `abandonStartup` with no screen.
The behavior D4 picks, add nothing when a cause is present, is still right, since most delivery causes already end with a tail and a second one would duplicate it.
The stated premise is false, though, and "get no tail on purpose" reads as a requirement a plan writer could encode as a test that delivery failures carry no pane lines.

**Fix:** Rewrite the sentence so it states what holds: a busy or unlanded send cause already ends with the pane's last lines, and the remaining causes (the reed-status retry exits, the skill-loader lookup, the prompt-offset persist and a raw key or paste delivery error) report a mechanism failure with no screen, so no cause gets a second tail.

### [LOW] D4's trigger sentence contradicts its own scope

**Location:** `_lyx/discussion/decision-record.md` D4, "The trigger is never-ready, not a nil cause".

**Issue:** The opening sentence scopes the tail to `awaitStartup`'s exits where the provider never reached `StartupReady` and `abandonStartup` gets no cause, which matches the code: the nil-cause calls at the pane-died, deadline and tick-cap exits of `awaitStartup`, and not the nil-cause skill-load death in `loadSkillsThenPrompt`.
The acceptance criterion picks the same set, so the decision and the criterion agree.
The second sentence then says the trigger is never-ready rather than a nil cause, but the three reed-status retry exits of `awaitStartup` are never-ready and carry a cause, and the record gives them no tail.
Taken alone, the sentence would put a tail on them.

**Fix:** Say the trigger is never-ready with no cause, and keep the skill-load death as the example of a nil cause that is not never-ready.

### [MEDIUM] D6's played-dismissal rule leaves a partial play unthrottled

**Location:** `_lyx/discussion/decision-record.md` D6, "Only a play that sent a non-empty sequence without error starts the gap; a failed play or an empty sequence leaves the next probe free to dismiss."

**Issue:** `playInputs` in `internal/shuttleengine/run.go` sends the sequence one key at a time and returns on the first failing `SendKey`, so a failed play can have delivered the arrow keys and not the Enter, or delivered some keys and failed later.
Those keys reached the pane, yet under D6 the play starts no gap, and the next probe one second later may play a fresh sequence computed from a capture taken before Claude redrew.
That is the stray-keypress hazard D6's rationale says the gap preserves, now reachable on a one-second cadence instead of the ten-second one it replaces.
Under the attack-surface rule, the exception lets a dismissal through and states no bound.
The other boundaries the focus names are stated: the multiple is floored to 1 as `Wait` floors it, a probe exactly one gap after the last play may dismiss, and `Wait`'s liveness ticks sit exactly one gap apart on its clock, so the at-or-past rule never holds a dismissal there.

**Fix:** Start the gap on any play that sent at least one key, failed or not, and leave it unstarted only for an empty sequence or a play whose first key failed; or state the bound that makes a partial play harmless and add an acceptance row for it.
