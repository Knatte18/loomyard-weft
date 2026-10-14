# Card 6 — claude-launch-skill-load

**Edit:**
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#TestPrepare_SkillsDeferPromptPointer`
- `internal/shuttleengine/claudeengine#TestPrepare_PromptLaunchLimit`
- `internal/shuttleengine/claudeengine#maxLaunchPromptBytes`
- `internal/shuttleengine/claudeengine#buildLaunchCmd`
- `internal/shuttleengine/claudeengine/doc.go`
- `contracts/stencils/roleskills_test.go`
- `PATTERN.md`
- `docs/overview.md`

**Intent:** Put Claude's skill-load message on the launch line when a spec names skills (D2, engine side), and reword `PATTERN-role-skills-typed` with it.

`Prepare`, for a spec with skills, builds `launchArg` from `c.SkillLoadMessage(spec.Skills)` in place of the empty argument, returns the prompt pointer as `PromptLine` as today, and sets `LoadsSkills: true`; a spec without skills keeps the pointer on the line with `LoadsSkills` false.
The message is one line already; `Prepare` refuses a message longer than `maxLaunchPromptBytes` before writing any artifact, with an error naming the length, the limit and the skill count, the same way it refuses an over-long pointer.
A resume spec takes the same rule, since `buildLaunchCmd` carries the first message on both its fresh and its `--resume` form.
The message text and the launch argument stay in `claudeengine` (`PATTERN-shuttle-provider-seam`).
Comments the change makes wrong, reworded here: `maxLaunchPromptBytes`'s comment says the argument is the pointer or, for a spec with skills, the one-line load message, either far under the bound; `buildLaunchCmd`'s comment and its `pointer` parameter name describe the first message as the pointer or the load message, no longer "none otherwise"; and `prepare_test.go`'s file comment names the launch-carried load message where it names the skills-deferred prompt pointer.

Tests: `TestPrepare_SkillsDeferPromptPointer` asserts, for a fresh and a resumed launch with skills, that `Launch.Cmd` carries the quoted `SkillLoadMessage(skills)` and no pointer and `LoadsSkills` is true, and without skills that `LoadsSkills` is false; its doc comment says so.
`TestPrepare_PromptLaunchLimit` (`settings_test.go`), which owns the `maxLaunchPromptBytes` refusal, gains a subtest beside `PathologicalRunDir_RejectedBeforeArtifacts` whose spec names enough long skill names that the load message exceeds `maxLaunchPromptBytes`, asserting the refusal names the launch limit and no artifact is written.
`TestStart_LoneEOLMarkBoxTakesStartupSends` and the corpus replay keep their canned launches without `LoadsSkills`, so they still pin the typed path card 5 keeps.

`PATTERN.md`'s entry becomes, verbatim (the same byte length as today's line, since the overview sits at its headroom line):

```markdown
- `PATTERN-role-skills-typed` — Loading a spawn's skill: the spawning module names it on the launch spec and lyx delivers it, on the launch line or typed; no stencil asks for one. (test)
```

`roleskills_test.go` is listed whole because both its file comment and a test body change: the file comment says lyx delivers a named skill on the launch line or typed, and `TestStencils_NoSkillLoadInstructions`'s failure message says "a skill is named on the launch spec and delivered by lyx" in place of "typed by lyx".
`docs/overview.md`'s skills line says a spawn's skills are delivered by the provider engine, on the launch line or typed, before the prompt.
`claudeengine/doc.go` replaces "starts on an empty input box" with the launch line carrying the load message and `LoadsSkills`.

**ImpactSummary:** Changes Claude's launch line for skill-naming specs only; shuttle handles the new field since card 5; PATTERN entry reworded at equal length; docs and one test message follow.
