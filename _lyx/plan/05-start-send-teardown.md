# Card 5 — A start-time send that does not land is a not-started run

**Edit:**
- `internal/shuttleengine#Run.abandonStartup` -> `func (run *Run) abandonStartup(outcome Outcome, cause error) (Result, error)`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#ErrNotStarted`
- `internal/shedengine#TransientAgentStart`
- `internal/shuttleengine/startup_test.go`

**Uses:**
- `plan:internal/shuttleengine#ErrSessionBusy`
- `plan:internal/shuttleengine#ErrSubmissionNotLanded`
- `plan:internal/shuttleengine#eventsSize`
- `plan:internal/shuttleengine#turnStartedSince`

**Intent:**
Implement D4: no start error past `AddStrand` leaves a live strand behind a record reading `running`.

`abandonStartup` takes the cause of the teardown: nil for awaitStartup's not-ready answers (its three existing calls pass nil and keep today's error text), or the send or persist error that ended the start.
When the cause is non-nil its returned error wraps both `ErrNotStarted` and the cause (`%w` twice), so `errors.Is` matches `ErrNotStarted` and the send's own sentinel, and the message still names the run dir, strand, outcome, capture note and removal note.
Its single return statement keeps its one `Errorf`, so the completion-signal tripwire's audited set is unchanged.

`loadSkillsThenPrompt` routes every failure through `abandonStartup(OutcomeDied, err)`: a skill-load send that fails, the prompt-offset persist that fails, and a prompt delivery that fails.
Before the prompt delivery it records `eventsSize(run.state.EventsPath)`; when the delivery fails with `ErrSubmissionNotLanded` it reads the session signals once more, and a `turn_start` past that offset (`turnStartedSince`) means the prompt landed late, so start returns the run handle instead of tearing down.
The box is not re-read there, since the send's own clear has emptied it.
The residual, stated in the method's doc comment: a prompt consumed between that signal read and the strand removal is killed with its agent; if it wrote an output file, the re-step's `Spec.validate` refuses on that file.

`ErrNotStarted`'s doc comment widens to a provider that never became ready or never took its first input, and `TransientAgentStart`'s doc comment widens to "an agent session that never became ready or never took its first input"; the class set and `shedtransient`'s mapping stay as they are (`PATTERN-transient-stop`: no new class, no new marking site).

Bound: the widened class still excludes an agent that took its prompt and then died or timed out; `ErrSessionBusy` typed nothing, and an unlanded prompt is caught twice (card 4's pre-clear reading, then this signal read) before the teardown.

Tests: `TestStartup_SkillLoadTurns` (or `TestStartup_RunGatedMappings`, whichever tables start's outcomes) gains rows on a `fakeClock`: a skill-load send that stays busy and a prompt delivery that does not land each return an error matching both `ErrNotStarted` and the send's sentinel, with the run's record `died` and its strand removed; a prompt-offset persist failure (run dir made unwritable after the load turn) takes the same teardown; and a prompt delivery that fails `ErrSubmissionNotLanded` while a `turn_start` lands past the pre-send offset starts the run.
`RunGated`'s mapping row shows the teardown as a `NotStarted` died result with a nil error.

**ImpactSummary:** `abandonStartup` is re-signed with both callers (`awaitStartup`, `loadSkillsThenPrompt`) on this card; start's failure paths past `AddStrand` now all tear down, and two doc comments widen with no behaviour change outside shuttle.

**Commit:** 5: shuttle: a start-time send that does not land tears the run down as not started
