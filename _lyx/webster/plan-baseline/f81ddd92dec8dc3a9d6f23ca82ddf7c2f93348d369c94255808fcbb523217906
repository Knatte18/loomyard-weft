# Card 5 — A start-time send that does not land is a not-started run

**Edit:**
- `internal/shuttleengine#Run.abandonStartup` -> `func (run *Run) abandonStartup(outcome Outcome, cause error) (Result, error)`
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#ErrNotStarted`
- `internal/shuttleengine#Runner.RunGated`
- `internal/shedengine#TransientAgentStart`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/completionsignal_enforcement_test.go`

**Uses:**
- `internal/shuttleengine#ErrSessionBusy`
- `internal/shuttleengine#ErrSubmissionNotLanded`
- `internal/shuttleengine#eventsSize`
- `internal/shuttleengine#turnStartedSince`

**Intent:**
Implement D4: no start error past `AddStrand` leaves a live strand behind a record reading `running`.

`abandonStartup` takes the cause of the teardown: nil for a provider that never became ready, or the error that ended the start.
Its nil callers keep today's error text: `awaitStartup`'s three not-ready answers (died, deadline, tick cap) and `loadSkillsThenPrompt`'s died load turn.
`loadSkillsThenPrompt`'s other existing call, the `skillLoader` failure, passes the loader's error, which it drops today.
When the cause is non-nil its returned error wraps both `ErrNotStarted` and the cause (`%w` twice), so `errors.Is` matches `ErrNotStarted` and the send's own sentinel, and the message still names the run dir, strand, outcome, capture note and removal note.
Its single return statement keeps its one `Errorf`, so its own audited entry is unchanged.
Its doc comment widens from "awaitStartup's not-ready teardown" to start's teardown for a provider that never became ready or never took its first input, and its closing Warn says which: "provider never became ready" for a nil cause, "start failed before the provider took its first input" with the cause otherwise.

`awaitStartup`'s three status-cap exits (`errStrandNotTracked`, `errStrandPaneBindingCleared`, and reed status failing `maxStatusRetries` times) are start failures past `AddStrand` that today return `run.identity()` with the record reading `running` over a strand reed could not account for.
Each now returns `run.abandonStartup(OutcomeDied, <its existing Errorf>)`, behind the `allOutputFilesExist` check that already precedes them, so the record is `died`, the strand removal is attempted and the error wraps `ErrNotStarted` beside its own sentinel, which routes it to the agent-start transient class as D4 decides for every start failure.
For an untracked strand the removal may itself fail, and `abandonStartup`'s existing removal note then names that, as for any failed removal.
`awaitStartup`'s return-mapping doc comment drops the "(run.identity(), err) without ErrNotStarted ... tears nothing down" case, and `RunGated`'s doc comment drops "a startup mechanism failure" from the errors it returns unchanged, since those now reach it as a `NotStarted` result.
In `completionsignal_enforcement_test.go`, the audited key `awaitStartup [Errorf]` x3 becomes `awaitStartup [Errorf,OutcomeDied]` x3, its justification line saying each is the status-cap teardown behind the direct `allOutputFilesExist` check.

`loadSkillsThenPrompt` routes every failure through `abandonStartup(OutcomeDied, err)`: a skill-load send that fails, the prompt-offset persist that fails, and a prompt delivery that fails.
Before the prompt delivery it records `eventsSize(run.state.EventsPath)`; when the delivery fails with `ErrSubmissionNotLanded` it reads the session signals once more, and a `turn_start` past that offset (`turnStartedSince`) means the prompt landed late, so start returns the run handle instead of tearing down.
The box is not re-read there, since the send's own clear has emptied it.
The residual, stated in the method's doc comment: a prompt consumed between that signal read and the strand removal is killed with its agent; if it wrote an output file, the re-step's `Spec.validate` refuses on that file.

`ErrNotStarted`'s doc comment widens to a provider that never became ready or never took its first input, and `TransientAgentStart`'s doc comment widens to "an agent session that never became ready or never took its first input"; the class set and `shedtransient`'s mapping stay as they are (`PATTERN-transient-stop`: no new class, no new marking site).

Bound: the widened class still excludes an agent that took its prompt and then died or timed out; `ErrSessionBusy` typed nothing, and an unlanded prompt is caught twice (card 4's pre-clear reading, then this signal read) before the teardown.
The status-cap exits now reach the transient class too, but only before the provider was seen ready, so no prompt was delivered, and a re-step they license is the same one a not-ready start already licenses today.

Tests: `TestStartup_SkillLoadTurns` (or `TestStartup_RunGatedMappings`, whichever tables start's outcomes) gains rows on a `fakeClock`: a skill-load send that stays busy and a prompt delivery that does not land each return an error matching both `ErrNotStarted` and the send's sentinel, with the run's record `died` and its strand removed; a prompt-offset persist failure (run dir made unwritable after the load turn) takes the same teardown; and a prompt delivery that fails `ErrSubmissionNotLanded` while a `turn_start` lands past the pre-send offset starts the run.
`RunGated`'s mapping row shows the teardown as a `NotStarted` died result with a nil error.
`TestStartup_MechanismFailure` pinned the status-cap exits tearing nothing down; its assertions are corrected rather than a test added: each row now asserts an error wrapping both `ErrNotStarted` and its own sentinel, one strand removal, and a record reading `died`, and its doc comment says so.

**ImpactSummary:** `abandonStartup` is re-signed with both calling functions (`awaitStartup`, `loadSkillsThenPrompt`) on this card; start's failure paths past `AddStrand`, the status-cap exits included, now all tear down and wrap `ErrNotStarted`, which outside shuttle changes only the class a status-cap start failure carries at the shed boundary.

**Commit:** 5: shuttle: a start-time send that does not land tears the run down as not started
