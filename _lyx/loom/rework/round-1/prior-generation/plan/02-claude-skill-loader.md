# Card 2 — Claude realizes skill loads as typed slash commands

**Edit:**
- `internal/shuttleengine/claudeengine/session.go`
- `internal/shuttleengine/claudeengine/claudeengine.go`
- `internal/shuttleengine/claudeengine/command.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine/claudeengine/session_test.go`
- `internal/shuttleengine/claudeengine/prepare_test.go`
- `internal/shuttleengine/claudeengine/command_test.go`

**Uses:**
- `internal/shuttleengine#SkillLoader`
- `internal/shuttleengine/claudeengine#launchPointer`
- `internal/shuttleengine/claudeengine#Claude.ClearSessionSequence`

**Intent:**
Make `*Claude` satisfy `SkillLoader` (card 1), with the slash syntax living only here, beside `ClearSessionSequence` and `CompactSessionSequence` in `session.go`.

- `SkillLoadSequence(skill)` returns `/<skill>` typed and submitted, with no leading Escape, for the same reason as `ClearSessionSequence`.
- `SkillUnknown(capture, skill)` reports whether the capture, in `normalizeCapture` form, shows Claude Code's unknown-command notice naming the skill (`Unknown skill: <name>` or `Unknown slash command: <name>`), with no running-turn hint.
  The needles are package constants beside the sequence.
  Claude Code's exact reaction to an unknown skill is unverified (decision record, Open risks): the doc comment states that a provider that instead treats the line as a plain prompt ends a turn, which confirms the load, and one that shows nothing is skipped at the timeout, so every reaction is bounded.
- `DefaultSkillLoadTimeout` returns a `defaultSkillLoadTimeout` constant beside the sequence; two minutes, since a skill load is one short model turn.

`Prepare` changes only for a spec with a non-empty `Skills` list: the launch line carries no prompt pointer (a fresh launch starts on an empty input box; a `--resume` launch resumes with no new message), and `Launch.PromptLine` is `launchPointer(promptPath)`.
`buildLaunchCmd` takes the pointer as a possibly-empty argument instead of building it, so both shapes share one composer; `prompt.md` is still written either way.
An empty `Skills` list produces exactly today's launch line and an empty `PromptLine`.

Tests: the sequence's text and submit flag; `SkillUnknown` true on both notice forms naming the skill, false on a notice naming another skill, on an idle box and on a running turn; `Prepare` with skills omits the pointer from `Cmd` and sets `PromptLine`, and without skills keeps the pointer and leaves `PromptLine` empty, for a fresh and a resumed launch.
The package doc names the skill-load sequence beside the clear and compact sequences.

**ImpactSummary:** claudeengine only; the no-skills launch line is unchanged, so every existing caller and its command-line pins stay green.

**Commit:** 2: claudeengine: load skills as typed slash commands and defer the prompt pointer
