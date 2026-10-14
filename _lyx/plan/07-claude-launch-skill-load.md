# Card 7 — claude-launch-skill-load

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestPrepare_RejectedBeforeArtifacts`
- `internal/shuttleengine/claudeengine#cannedLaunchEngine.Prepare`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`
- `internal/shuttleengine/claudeengine_test#replayCorpusCase`
- `internal/shuttleengine/claudeengine/doc.go`
- `PATTERN.md`
- `docs/overview.md`
- `contracts/stencils/roleskills_test.go`

**Intent:** D2, Claude's side: a spec that names skills launches with `SkillLoadMessage(spec.Skills)` as the launch argument in place of today's empty input box, keeps the prompt pointer as `Launch.PromptLine`, and sets `Launch.SkillLoadOnLaunch`, for a fresh and a resumed launch alike.
A spec without skills launches with the prompt pointer as today, with the flag false.
The message text stays in `skillload.go` and every Claude-specific detail under `claudeengine` (`PATTERN-shuttle-provider-seam`).

`Prepare` checks the message's length against `maxLaunchPromptBytes` beside the pointer check, before any artifact is written, and refuses an over-long one with an error naming its byte count, the limit and that the spec names too many or too long skills.
The input that reaches it: a spec whose skill list renders a message over 30000 bytes, which no upstream check bounds (`Spec.validate` does not inspect skill names); `TestPrepare_RejectedBeforeArtifacts` gains the row `skill_load_message_over_the_launch_limit` with `Skills: []string{strings.Repeat("s", 30001)}`, `cfg: templateConfig(t)` (so the prompt-cache TTL resolves and the length check is reached), `wantErrContains: "skill-load message"`, and its existing no-artifacts assertion.
The message is one line, as `TestSkillLoadMessage_IsOneSlashlessLineNamingEverySkillInOrder` already pins.

Tests:
- `TestPrepare_SkillsDeferPromptPointer`, for both resume values: with skills, `Launch.Cmd` holds the load message's text (`Load these skills now` and the skill name) and no `Read ` pointer, `PromptLine` is the pointer and `SkillLoadOnLaunch` is true; without skills, the pointer is on `Cmd`, `PromptLine` is empty and the flag is false.
  Its doc comment says the launch line carries the load message.
- `cannedLaunchEngine.Prepare` returns `SkillLoadOnLaunch: true` beside its `PromptLine`.
  `TestStart_LoneEOLMarkBoxTakesStartupSends` asserts the pane received `/color blue` and `Read the prompt.` and never the skill-load message, the launch line having carried it; its events file gets no turn end, so the load turn ends at the skill-load timeout on the virtual clock, and the color and prompt still follow.
- `replayCorpusCase` sets `launch.SkillLoadOnLaunch = true` where it sets `PromptLine` for a case with skills, so the replay's first line is read as the launch-carried load turn's end.

Docs, in the same commit:
- `claudeengine/doc.go`: the sentence saying a spec that names skills starts on an empty input box becomes: the launch line carries `SkillLoadMessage` as the session's first turn, with `Launch.SkillLoadOnLaunch` set and the pointer returned as `Launch.PromptLine`; `Prepare`'s doc comment matches.
- `PATTERN.md`'s `PATTERN-role-skills-typed` entry becomes, verbatim:

```markdown
- `PATTERN-role-skills-typed` — Loading a skill into a spawned session: the spawning module names it on the launch spec and lyx delivers it, on the launch line or typed; no stencil asks an agent to load a skill. (test)
```

- `docs/overview.md`'s skills bullet, a different line from the loom lines card 4 edited, says a spawn's skills are named on the launch spec and delivered by the provider engine before the prompt, on the launch line or typed.
- `contracts/stencils/roleskills_test.go`'s file comment says lyx delivers the skill, on the launch line or typed, in place of "lyx types it".

**ImpactSummary:** Claude's launch line for a skills spec changes from empty to the load message with a new length refusal; shuttle's card 6 path consumes it, and the three claudeengine tests that build a skills launch change with it; PATTERN, overview and one test comment are reworded.

**Commit:** 7: Carry Claude's skill-load message on the launch line
