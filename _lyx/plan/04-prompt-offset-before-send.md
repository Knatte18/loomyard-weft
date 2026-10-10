# Card 4 — prompt-offset-before-send

**Edit:**
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#TestStartup_SkillLoadTurns`

**Intent:** The second half of D4: `PromptOffset` is the events-file size read immediately before the prompt is sent, after the load turns, so anything the provider appended after the last load turn end and before the prompt lies before it (AC12).

In `loadSkillsThenPrompt`, after the load turns, read the events file's size once through `eventsSize`; when it is positive, set `run.offset` and `run.state.PromptOffset` to that reading and persist the record as today, and reuse the same reading as the `sentAt` the late-landing check consults, so there is one read, not two.
The `run.offset > 0` guard becomes a guard on the size reading, which keeps a launch without skills, whose file is empty, persisting nothing, as today.
The doc comment on `loadSkillsThenPrompt` and the `PromptOffset` field comment on `RunState` say the offset is the file size at the moment before the prompt is sent, past the load turns and any line appended after them.

`TestStartup_SkillLoadTurns` keeps its offset assertions and gains a row in which a line is appended to the events file after the last load turn end and before the prompt goes out, through the fake clock's step hook, asserting the persisted `PromptOffset` and `run.offset` equal the file's size including that line.
The package doc's sentence on the offset is rewritten by card 5 together with the hold guard.

**ImpactSummary:** One method reads the file size instead of its own offset; `RunState`'s field doc changes; readers of `PromptOffset` (`activity.go`, `sessionread.go`, `attach.go`, webster's recovery) read a value that can only be larger, and no signature changes.
