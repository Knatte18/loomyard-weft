# Card 1 — Shuttle launch spec names the skills a session loads

**Edit:**
- `internal/shuttleengine/spec.go`
- `internal/shuttleengine/engine.go`
- `internal/shuttleengine/run.go`
- `internal/shuttleengine/session.go`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/fakes_test.go`
- `internal/shuttleengine/startup_test.go`
- `internal/shuttleengine/session_test.go`

**Create:**
- `plan:internal/shuttleengine#SkillLoader` -> `type SkillLoader interface`
- `plan:internal/shuttleengine#Runner.LoadSkill` -> `func (r *Runner) LoadSkill(guid, skill string) error`
- `plan:internal/shuttleengine#Runner.SkillUnknown` -> `func (r *Runner) SkillUnknown(guid, skill string) (bool, error)`

**Uses:**
- `internal/shuttleengine#Run.awaitStartup`
- `internal/shuttleengine#SessionCycler`

**Intent:**
Add the provider-neutral half of D1: a spawning module names the skills a session loads before its prompt, and shuttle loads them on every fresh launch.

`Spec` gains `Skills []string` (provider-neutral skill names, in load order) and `SkillLoadTimeout time.Duration`.
`validate` does not inspect either; a zero timeout means the engine's default.
`Launch` gains `PromptLine string`: when set, the engine has left the prompt pointer off the launch line, and shuttle delivers this text as a turn once the skills are loaded.
An empty `Skills` list changes nothing: the engine returns no `PromptLine`, and the launch is byte-identical to today's.

A new optional capability `SkillLoader` sits beside `SessionCycler` and `SessionResumer`, for the same reason they are separate from `Engine`:
- `SkillLoadSequence(skill string) []PaneInput` — the key choreography that loads one skill as a submitted turn;
- `SkillUnknown(capture, skill string) bool` — whether a pane capture shows the provider reporting that skill unknown;
- `DefaultSkillLoadTimeout() time.Duration` — the engine-owned default bound.
A spec with a non-empty `Skills` list on an engine without `SkillLoader` is an error from `start`, before any run directory exists.

In `Runner.start`, after `awaitStartup` reports ready and only when `PromptLine` is set, the run loads each skill in order: it plays the skill's sequence, then polls (on the run's injectable clock, at the poll interval) until a turn end (`EventStop` or `EventWaiting`) is read from the events file past the offset taken before typing, the pane capture classifies `SkillUnknown`, or the skill-load timeout passes.
The two skip causes are logged through `internal/logger` with the skill name, the cause and the strand guid, and the run moves on; a skipped skill never fails or hangs the launch.
Then it delivers `PromptLine` through the verified send path `Runner.Send` already uses.
The run's own events read offset starts past every skill-load turn end, so `Wait` never classifies a skill load's turn end as the run asking.
A pane that dies during skill loading is a died startup, torn down as `abandonStartup` does today.

`Runner.LoadSkill` plays one skill's sequence into a live pane with no idle precondition of its own (the caller has probed idleness, as for `ClearSession`), and `Runner.SkillUnknown` captures the pane and asks the engine; both are the orch watcher's per-tick primitives (card 14), and both error naming `SkillLoader` when the engine lacks it.
Neither carries slash-command text: that stays in the provider engine (`PATTERN-shuttle-provider-seam`).

Tests, on the existing fake engine with an injected clock and a shortened timeout: skills load in order before the prompt is sent; an empty list launches with no extra input; a skill whose load ends no turn is skipped at the timeout and the next skill and the prompt still go; a skill the fake reports unknown is skipped at once; skill-load turn ends do not reach `Wait`'s classification; an engine without `SkillLoader` refuses a spec that names skills; `LoadSkill` and `SkillUnknown` delegate to the capability.
The package doc gains a short "Skill loading" section stating this contract.

**ImpactSummary:** additive fields and an optional capability inside shuttleengine; no caller sets `Skills` yet, so every existing launch is unchanged.

**Commit:** 1: shuttle: launch spec names the skills a session loads before its prompt
