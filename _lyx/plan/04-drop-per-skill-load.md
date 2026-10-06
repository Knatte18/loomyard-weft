# Card 4 — drop-per-skill-load

**Delete:**
- `internal/shuttleengine#SkillLoader.SkillLoadSequence`
- `internal/shuttleengine#SkillLoader.SkillUnknown`
- `internal/shuttleengine#Runner.LoadSkill`
- `internal/shuttleengine#Runner.SkillUnknown`
- `internal/shuttleengine/claudeengine#Claude.SkillLoadSequence`
- `internal/shuttleengine/claudeengine#Claude.SkillUnknown`
- `internal/shuttleengine/claudeengine#unknownSkillNeedle`
- `internal/shuttleengine/claudeengine#unknownCommandNeedle`
- `internal/shuttleengine/claudeengine#TestSkillLoadSequence`
- `internal/shuttleengine/claudeengine#TestSkillUnknown`

**Edit:**
- `internal/shuttleengine#Runner.skillLoader`
- `internal/testkit/shuttlefake/shuttlefake.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/session_test.go`
- `internal/shuttleengine/claudeengine/session.go`

**Intent:**
Remove the pane-text unknown-skill classifier and the per-skill `/<skill>` load now that no caller uses them (Decision 8).

`SkillLoader` keeps `SkillLoadMessage`, `ClassifySkillLoad` and `DefaultSkillLoadTimeout`; `Runner.LoadSkill` and `Runner.SkillUnknown` go, leaving `Runner.LoadSkills` and `Runner.ClassifySkillLoad`.
`Claude.SkillLoadSequence`, `Claude.SkillUnknown` and their two needles go; `claudeengine/session.go`'s file comment stops naming the unknown-skill classifier and points at `skillload.go` for the `SkillLoader` realization.
`shuttlefake.Engine` drops `SkillLoadSequenceFn`, `SkillUnknownFn` and their methods, and `skillFakeEngine` drops its two old methods.
`skillLoader`'s refusal names the capability's current methods (`SkillLoadMessage`, `ClassifySkillLoad`, `DefaultSkillLoadTimeout`); its tests assert only the substring `SkillLoader`, so no assertion changes.
`session_test.go`'s guard and plain-engine tests drop their `LoadSkill`/`SkillUnknown` assertions and are renamed for `LoadSkills`/`ClassifySkillLoad`.
`TestSkillLoadSequence` and `TestSkillUnknown` go with the code they pinned.

After this card no `SkillUnknown` pane classifier or `/<skill>` load line remains in production code, which `grep -rn 'SkillUnknown\|SkillLoadSequence' --include=*.go internal` confirms empty.

**ImpactSummary:** Deletes the old per-skill primitives across shuttleengine, claudeengine and shuttlefake; cards 2 and 3 already moved every caller off them.

**Commit:** `4: shuttle: drop the per-skill load and the pane unknown-skill classifier`
**Verify:** ! grep -rn 'SkillUnknown\|SkillLoadSequence' --include=*.go internal
