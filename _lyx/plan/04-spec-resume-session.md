# Card 4 — spec-resume-session

**Edit:**
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/command_test.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`

**Create:**
- `plan:internal/shuttleengine/claudeengine#validateSessionID` -> `func validateSessionID(id string) error`

**Intent:**
Add `ResumeSessionID string` to `Spec`, engine vocabulary: when set, the engine launches that existing session instead of minting a new one.
`validate` does not inspect it.

In `claudeengine`, add `validateSessionID`, which accepts only the lowercase hyphenated UUID shape `newSessionID` mints and rejects anything else with an error naming the expected shape.
It also keeps shell metacharacters out of the quoted id on the pane line.
When `spec.ResumeSessionID` is non-empty, `Prepare` validates it before any artifact is written and uses it as `Launch.SessionID` instead of calling `newSessionID`.
`buildLaunchCmd` then emits `--resume <id>` in place of `--session-id <id>`, keeping the positional prompt read from `prompt.md`, this run's own `--settings`, `--name`, `--model`, `--effort`, the permission flag from card 1, the notice and the fork env wrapping;
the run's own settings file is what routes the adopted session's Stop hook into this run's events file.
`buildResumeCmd` already names `Launch.SessionID`, so reed's resume line names the same id with no further change.
Pass the choice to `buildLaunchCmd` as a `resume bool` parameter, and update its doc comment to describe both shapes.

Tests: a spec with `ResumeSessionID` produces a launch line with `--resume` naming the id, no `--session-id`, and the prompt read, settings path, model, effort and `--dangerously-skip-permissions` under `bypass`;
its resume line names the same id;
`Launch.SessionID` equals the given id;
a malformed id (wrong length, uppercase, a quote character) fails `Prepare` before any artifact exists;
an empty `ResumeSessionID` keeps today's `--session-id` line.
Extend the `internal/shuttleengine` package doc paragraph to name `ResumeSessionID`.

**ImpactSummary:** Adds one Spec field and one branch in claudeengine's launch line; an empty field leaves every existing launch byte-identical.
