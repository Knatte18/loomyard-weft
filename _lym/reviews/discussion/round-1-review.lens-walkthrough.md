```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: each decision run end to end on one concrete case, checked against its own stated purpose.

**Decisions walked:**
- D1, a board `run_settings` holding `unattended` read by `WithRunAutonomy`, then the orch: purpose met.
- D2, board claim at `true` over halt counts 1, 2 and 3 (lym, then orch, then operator): purpose met. Dead driver row: purpose not met (F2). Permission dialog row: purpose not met (F3).
- D3, two lym re-steps at Worktree-Create and then a third stop: purpose met if settle records gain a halt-entry link. Today `SettleRecord` has no such field and is keyed by the full `StopKey` (producer, history length, state). The record states the link as a decision, so this holds.
- D4, a run handed back on a pause, then the pause lifts: purpose not met (F4). Gate-attempts re-step: weak (F6).
- D5, orch `lym loom resume` against a halted run whose driver is not yet parked: purpose met.
- D6, budget escalation under `unattended`, then continue, then the next escalation at the hard cap: purpose broken (F1). The arbiter's input lacks the level and the segment memory (F5).
- D7, third-halt severe verdict under `true`, with the dobby's orch slack button off: purpose not met (F7).
- D8, severe post with operator mention: purpose met. The dedup can suppress the mention on a second, different halt of the same slug within the interval (folded into F7's fix note).
- D9, unattended cancel turned into a pause and a held line: purpose met.
- D10, `settle: orch` halt left 10 minutes, then the chase, then the severe post: purpose met for halts the driver stamps. Dead driver: not met (F2).
- D11, Raddle-Write with a passing proposal and a live agent: purpose met.
- D12, `unattended` with the button off: purpose met for `unattended` only (F7).
- D13: no mechanism to walk.

**Findings:**

F1 — BLOCKING, design, `_lym/discussion/decision-record.md` D6 (and D2's hard-cap row)
D6's bound "nothing lands a round with an open BLOCKING finding under `unattended`" breaks through the arbiter's `hard-cap` accept.
- The arbiter still takes `hard-cap` "as today". The study shows it settles hard-cap by accept (17 of 20, study line 52), and `arbiterstop.Eligibility` admits hard-cap with no level check (`internal/arbiterstop/arbiterstop.go:120`).
- Walked case: under `unattended`, a budget escalation gets one `circling continue`. A round that still holds an open BLOCKING finding can then reach the perch's hard cap, where `circling continue` refuses (`internal/loomcli/circling.go:127`). That escalates as `hard-cap`, and the arbiter accepts it, landing the BLOCKING finding unattended.
- A hard-cap escalation reached directly with open BLOCKING findings does the same.
- Fix: D6 also binds the hard-cap accept under `unattended`: hand on, or pause and hold.

F2 — MEDIUM, design, D2 row "Dead driver, agent API error" and D10
A dead driver past its cap never reaches the chase or the severe path.
- D3 says these are no stop at a row and are reported only from the dobby's notices. D7 stamps `class:`/`settle:` only on halts "the driver hands back", and a dead driver hands nothing back.
- D10 chases only "a run whose stop record reads `settle: orch`" (decision-record.md:248).
- The study names dead drivers after a crash as the long overnight holds ("23 min to overnight", study line 92). D10's purpose of no run left halted until morning is missed for that class.
- Evidence that today's dobby reading carries no settle stamp: `ChildReading` (`internal/dobbyengine/runnotice.go:69-85`).

F3 — MEDIUM, design, D2 row "Provider permission dialog" and D9
No step hands the dialog to the orch.
- The support log (line 23) records that no mid-run permission dialog is detected.
- `internal/shuttleengine/claudeengine/session.go:79` reads a dialog's caret as not idle, so the quiet notice does not fire either.
- The orch's rule "reads and answers / denies when unsure" therefore receives no input. The walked end state is a run stalled behind the dialog with no notice, which is the study's 4–9 min hold, unbounded overnight.

F4 — MEDIUM, design, D4 pause resume and Notes line 390
D4 modifies a run-level resume line that does not exist.
- D4 says the dobby resumes "as the orch does today on the pause watcher's resume line; the line then says lym resumed the run".
- The pause watcher's line is `pauserecord.ResumeLine` (`internal/pauserecord/record.go:31`), given to a session a pause interrupted ("Carry on where you stopped…"). It is not a line telling the orch to resume runs.
- `queueResumeLine` does not exist anywhere in `internal/` (grep found no match).
- The dobby gives a halted or paused run no pass until a hand pass (`internal/dobbyengine/doc.go:51`). The `liftPending` path (`passes.go:210-213`, `361`) drops the lift for a run that is not drivable.
- How the dobby matches "that pause's hand-back" to a pause entry is also unstated.
- So the walked step "change the line" has no input, and the resume mechanism and its pause-entry match are left for Plan to find out.

F5 — LOW, design, D6 arbiter rules
The arbiter is handed neither the level nor whether the segment was already continued.
- `arbiterstop.Stop` carries no autonomy field (`arbiterstop.go:29-64`).
- The arbiter's verbs are `circling accept`/`continue` (`internal/loomcli/arbiterprompt.go:19`). D6 also has it run `lym board upsert`, but which of its settings allow that is not stated.
- As walked, "hands on under `unattended`" and "once per review segment" have no input that tells the arbiter either fact.

F6 — LOW, design, D4 gate-attempts row
A plain re-step with a fresh producer session gets the same gate and the same change, and is likely to exhaust again.
- The study marks gate attempts exhausted as judgment, settled 4 of 4 by the orch with `decision add` (study lines 61 and 138).
- The cited rationale (the tackler resumed 57 of 65) does not cover this class.
- As walked, the end state is one wasted attempt cycle before the orch, against D4's purpose of settling with a bound "without an agent".

F7 — MEDIUM, design, D7 with D12
Under `true` a severe halt can reach no one while the dobby records it as posted.
- The dobby posts as the scoped program `&lt;shortname&gt;:&lt;slug&gt;:DOBBY` (`internal/beamcli/dobbydeps.go:647`). The relay drops that mail at Info when the routed orch's button is off (`internal/slackengine/outbound.go:197-198`, `postTo` 290-292).
- The creel post itself succeeds, so D7's "posted" state is written and D8's reminders run to a dropped channel.
- D7 forbids the orch to relay a `settle: operator` halt, and D12 warns only under `unattended`. Under `true` with the button off, the operator is never reached and nothing says so.
- Side note on D8: the per-slug mention dedup within `slack.remind_after_min` also silences the mention for a second, distinct severe halt of the same slug, such as D10's post after a D7 post.

Tool calls:
1. `ChildReading` in `internal/dobbyengine/runnotice.go` and `SettleRecord` in `internal/shedrun/settle.go`, to see the fields the dobby reads and the settle record holds; my context had only the alias `sheddriver.SettleRecord`.
