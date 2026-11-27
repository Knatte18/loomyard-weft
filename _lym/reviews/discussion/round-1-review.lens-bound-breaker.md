```yaml
lens: bound-breaker
outcome: reported
```

Task: bound-breaker lens. I attacked each bound, guarantee and code claim in the decision record and looked for a counterexample.

**Claims attacked**
- D1, every check path accepts or refuses alike, and a refused value reads `false`: held.
- D1, the older-binary bound: held.
- D2, the verdict order makes every row read lym, then orch, then severe: broken (F4).
- D2 and D9, the provider-permission-dialog row: broken, nothing detects the dialog (F6).
- D3, each stop is counted once, and a handed-on escalation counts: broken, the counting is undefined (F4).
- D3, `wakeRestartGap` never restores a table bound: held, as long as the new bound reads the whole history.
- D4, the classes are told apart without matching message text: broken for every row but board claim (F7).
- D4, the pause lift resumes "as the orch does today on the pause watcher's resume line": broken, no such per-run line exists (F8).
- D5, the verb waits instead of refusing `driver_not_parked`: broken for the run-lock branch (F5).
- D5, batten's path does not wait: held.
- D6, "continues at most once per segment": broken, nothing enforces it (F9).
- D6, "nothing lands a round with an open BLOCKING finding under `unattended`": broken (F1).
- D7, a severe halt reaches the operator, and the state record marks it posted: broken under `true` with the slack button off (F2).
- D7, the dobby already posts as a program to `&lt;shortname&gt;:SLACK`: held.
- D8, one mention per halt: broken for a second, distinct halt (F10).
- D8, the mention notifies the operator: at risk from the relay's escaping (F11).
- D10, an orch that stalls cannot leave a run halted until morning: broken (F3).
- D10, a pause or a transition ends the chase: held.
- D11, a skip lasts only while its proof stands: broken by the probe's existing meaning (F12).
- D12, warns when the slack button is off or no operator is configured: held for `unattended`, absent for `true` (part of F2).

**Findings**

F1 — BLOCKING, design, D6 and D9 (decision-record.md:181-182)
- Summary: the bound "nothing lands a round with an open BLOCKING finding under `unattended`" breaks through the arbiter's `hard-cap` accept, which D2 keeps "as today".
- Counterexample: `arbiterstop.Eligibility` admits `hard-cap` (internal/arbiterstop/arbiterstop.go:117). At the hard cap the only verbs are accept or raising the cap (internal/loomcli/circling.go:127). The study records 17 arbiter accepts for `hard-cap` (stop-classes doc:52). So under `unattended` a hard-cap round with open BLOCKING findings is accepted with no orch involved. The same holds for `circling` and `stalemate` accepts.

F2 — BLOCKING, design, D7, D8 and D12 (decision-record.md:193-204, 285-289)
- Summary: under `true` with the orch's slack button off, a severe halt reaches no one, while the record treats it as delivered.
- Counterexample: the relay drops the mail of an orch whose button is off and still marks it handled (internal/slackengine/outbound.go, `postMessages`, the `!sortedContains(forwarding, orch)` branch).
- The dobby's state record marks the halt posted once the creel post succeeds (D7).
- D7 forbids the orch to relay a notice that reads `settle: operator`, so an operator watching the orch pane never sees it either.
- D12 warns only for `unattended`, so nothing warns under `true`.
- D8's reminders are dropped the same way.

F3 — BLOCKING, design, D10 (decision-record.md:248-260)
- Summary: D10's chase keys on a stop record that reads `settle: orch`. Two stalls the record names in D10's rationale leave no such record, so an orch that stalls still leaves them until morning.
- Counterexample 1: an agent's question an orch never answers. D11 suppresses the idle notice while the question is open. The creel chase goes back to the same stalled orch. No stop record exists, so D10 never fires (study: 222 min, `board-readme-golden-tests`).
- Counterexample 2: a dead driver past its cap. D3 says it "is no stop at a row", and D2 gives it no operator cell. Neither D7 nor D10 ever reaches the operator (study: dead drivers held 23 min to overnight).

F4 — MEDIUM, design, D2 and D3 (decision-record.md:71-75, 106-116)
- Summary: D3 says an escalation the arbiter hands on "counts", but its counting rule reads only Stuck entries with no `routed_to` and settle records. A handed-on escalation leaves an Awaiting entry and no settle record, so whether it counts is undefined.
- If it counts across classes, two routine `decision` escalations handed on at Code-Review make the next unrelated stop at that row, a launch failure say, a severe halt at once. That skips lym's fresh start and wakes the operator.
- Evidence: internal/reportengine/timingwaits.go:88 tells Awaiting apart from a Stuck with no `routed_to`; role-autonomy.md:42 counts only the latter.

F5 — MEDIUM, design, D5 (decision-record.md:152-158)
- Summary: D5 keeps the run-lock refusal on a non-running run "at once, as today". That refusal carries the same `driver_not_parked` kind and the same "again in a few seconds" text the orch looped on, so the orch's retry loop survives for that branch.
- Evidence: internal/loomcli/resume.go, the `runLockHeld` branch refuses with `shedrun.StartNotParkedKind`.

F6 — MEDIUM, design, D2 and D9, permission-dialog row (decision-record.md:91, 237)
- Summary: the row gives the orch a dialog to "read and answer", but nothing notices a dialog, and the record never says how one would be noticed. This is a fact left for later research.
- Counterexample: `IdleSession` reads a permission or select dialog as not idle (internal/shuttleengine/claudeengine/session.go:79). The quiet notice therefore never fires for a session stuck behind one.
- Autonomous runs refuse every permission mode but skip-permissions (internal/shuttleengine/claudeengine/command.go:59).
- The support log says no mid-run dialog is detected (support-log.md:23).

F7 — MEDIUM, design, D4 (decision-record.md:130-136)
- Summary: only the board-claim row says how its class is told apart without message text. The rows for launch failure, session timeout, fixer fork failure, lost strand and gate attempts exhausted never say what field identifies them, and today's table matches only kind, transient class and two reason prefixes (internal/sheddriver/driverules.go header, `decideDrive`).
- Without a stated identifier, a broad match retries stops that are not that class, and the bound of one holds only per misread class.

F8 — MEDIUM, consistency, D4 and the Notes (decision-record.md:139-140, 390)
- Summary: "as the orch does today on the pause watcher's resume line; the line then says lym resumed the run" rests on a line that does not exist.
- Evidence: `queueResumeLine` exists nowhere in the code. The only resume line is `pauserecord.ResumeLine`, the line an interrupted session gets, about its own turn (internal/pauserecord/record.go:31).
- The orchengine pause watcher posts only the orch's own resume line (internal/orchengine/pause.go:3).
- The dobby's doc says a paused run gets no pass until the orch or the operator resumes it (internal/dobbyengine/doc.go:51), so no per-run line exists to change.

F9 — MEDIUM, design, D6 (decision-record.md:174-182)
- Summary: "continues at most once per segment" and "hands on after that round" need a memory that the arbiter already continued, and the record names none.
- Each escalation spawns a fresh arbiter, and `Eligibility` is stateless (internal/arbiterstop/arbiterstop.go:79), so a second `budget` escalation is admitted and can be continued again. Only the hard cap bounds it.
- The case of a missing or stale fixer report is undefined too.

F10 — MEDIUM, design, D8 (decision-record.md:213)
- Summary: the dedup "no mention for a second post for the same slug within `slack.remind_after_min`" also suppresses a distinct second severe halt of the same run. An example is a severe halt that is answered and resumed, then a new severe halt within the interval: the operator's phone is not notified of the second.

F11 — LOW, design, D8 (decision-record.md:211)
- Summary: the relay escapes the whole body (`slackEscape.Replace(message.Body)` in internal/slackengine/outbound.go, `postMessages`).
- A mention added before that escape becomes `&amp;lt;@U…&amp;gt;` and notifies no one, and the record does not say where the mention goes relative to the escape.

F12 — MEDIUM, design, D11 (decision-record.md:269-276)
- Summary: D11 has every runner's held notice read "that question" through the existing probe.
- `creelengine.Store.AwaitsReply` also answers true while the strand holds an unread reply (internal/creelengine/verbs.go:1083-1101).
- So the held notice is also suppressed for an agent that ended its turn without reading a reply, the case D11 itself says is no wait.
- The record does not say the probe must be split.

Tool calls:
1. Read `AwaitsReply` in `creelengine/verbs.go`, the autonomy-key skip in `landingshed/finalize.go`, the writers of `routed_to`, and `IdleSession`'s dialog handling in `claudeengine/session.go`. My context had only the names and call sites of these, not their meaning.
