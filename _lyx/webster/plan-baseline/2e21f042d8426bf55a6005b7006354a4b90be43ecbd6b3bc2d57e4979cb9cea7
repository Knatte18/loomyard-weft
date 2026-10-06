# Card 2 — launch-skills-one-turn

**Edit:**
- `internal/shuttleengine#SkillLoader`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#Run.loadSkill`
- `internal/shuttleengine#Spec`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/testkit/shuttlefake/shuttlefake.go`
- `internal/loomcli#integrationWriteStubDriverScript`
- `internal/loomcli#writeStubDriverScript`

**Create:**
- `plan:internal/shuttleengine#Runner.LoadSkills` -> `func (r *Runner) LoadSkills(guid string, skills []string) error`
- `plan:internal/shuttleengine#Runner.ClassifySkillLoad` -> `func (r *Runner) ClassifySkillLoad(turnEnd Event, skills []string) (SkillLoadReport, error)`

**Prosa:**
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#SkillLoadReport`
- `internal/shuttleengine/claudeengine#Claude.SkillLoadMessage`
- `internal/shuttleengine/claudeengine#Claude.ClassifySkillLoad`
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine#Runner.skillLoader`

**Intent:**
Switch shuttle's launch from one turn per skill to one load turn for the whole list, with at most one retry, and add the per-list `Runner` primitives the orch watcher moves onto in card 3 (Decisions 3, 5, 6, 7, 9, 10, 14; issue #394 on the launch side).

`SkillLoader` gains `SkillLoadMessage(skills []string) string` and `ClassifySkillLoad(turnEnd Event, skills []string) SkillLoadReport`.
`SkillLoadSequence` and `SkillUnknown` stay on it, unused by the launch, until card 4 removes them with their last callers.
The type comment describes the capability as list-shaped and `DefaultSkillLoadTimeout` as the bound on one load turn.

`loadSkill` becomes one load turn: it sends `loader.SkillLoadMessage(skills)` through `sendVerified` (Decision 3), then polls the events file until the first `EventStop` or `EventWaiting` past the run's offset, which it returns, the pane dies, or the timeout passes.
It no longer captures the pane for an unknown-skill notice.
It may be renamed for its new shape (e.g. `loadSkillTurn`); the run's offset still moves past every event read, so it ends past every load turn end, the retry's included (Decision 10).

`loadSkillsThenPrompt` runs the first turn over `spec.Skills` and acts on its outcome:

- timed out: every requested skill is skipped with `logger.Warn` cause `timeout`, and there is no retry (Decision 7);
- `Verified` false: one `logger.Warn` `skill load unverified` listing the skills, and no retry (Decision 6); an unknown skill on such a turn goes undetected and shows only in that Warn's list, which never stalls or fails the launch;
- otherwise each `Unknown` skill is skipped with a Warn naming it and cause `unknown`, and a non-empty `Missing` list gets exactly one retry turn naming only those skills.

The retry turn is classified the same way: timeout and unverified as above, `Unknown` with cause `unknown`, and whatever is still `Missing` with cause `not loaded`.
Then the prompt pointer goes out through `sendVerified`, as today.
A pane that dies in either turn is a died startup.
Bound: at most two load turns per launch, each bounded by the load timeout; a skipped skill never fails or hangs the launch, and a skill that loaded before a timed-out turn is logged as skipped but stays loaded.

`Runner.LoadSkills(guid, skills)` checks the capability, `FindRun` and `requireLiveStrand` like the old `LoadSkill`, then sends `SkillLoadMessage(skills)` through `sendVerified`, with no idle precondition of its own.
`Runner.ClassifySkillLoad(turnEnd, skills)` returns the engine's classification, or the `skillLoader` error on an engine without the capability.

The `Spec.Skills` and `Spec.SkillLoadTimeout` comments say the list loads in one turn and the timeout bounds one load turn (the first and the retry), keeping both names and defaults.
`shuttleengine/doc.go`'s skill-loading paragraph describes the one load turn, the classification, the retry, the unverified degrade and the timeout skip, and names `Runner.LoadSkills` and `Runner.ClassifySkillLoad` as the per-tick primitives.

`shuttlefake.Engine` and `skillFakeEngine` implement the two new methods: `skillFakeEngine.SkillLoadMessage` types a recognisable `LOAD:<a,b>` line and `ClassifySkillLoad` answers from a per-turn script; `skillReed` appends a turn end on a typed load line unless that turn is scripted to hang.
`startup_test.go` extends its existing skill tests rather than adding parallel ones, with rows for: all skills loaded in one turn before the prompt, with the offset past the turn end;
an unknown skill skipped at once with no retry and no wait;
a missing skill retried once in a message naming only it, then the prompt, with the offset past both turn ends;
still missing after the retry, skipped;
an unverified first turn and an unverified retry, each confirming with no further turn;
a first turn that never ends, skipped at the timeout with no retry, then the prompt;
a pane that dies during a load turn, a died startup.
The existing empty-list and refused-without-`SkillLoader` tests stay.
`session_test.go` extends the guard and plain-engine tests to `LoadSkills` and `ClassifySkillLoad`.

The two loomcli stubs (Decision 14) treat the first typed line as the load message: each takes the `--settings` path from its own arguments and appends `{"hook_event_name":"Stop","last_assistant_message":"ok"}`, with no `transcript_path`, to `events.jsonl` beside it, then reads the next line as the prompt pointer, as today.
Only each stub's skill-load answer and its comment change, since `test-suite-prune-core` edits `internal/loomcli` tests in parallel.

**ImpactSummary:** Changes shuttle's launch skill step and the `SkillLoader` capability; its implementers are claudeengine (card 1), shuttlefake and the package test fakes, and the two loomcli tmux stubs follow the new protocol.

**Commit:** `2: shuttle: load a launch's skills in one turn, retry missing once`
