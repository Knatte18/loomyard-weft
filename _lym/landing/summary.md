# Render every producer's prompt and edit directive for its launch engine

Before this change, only the assist seats rendered the edit directive in the launch engine's own wording.
Every other producer rendered the Claude-worded shared directive on any engine,
and both engine copies lacked rules the shared one had.
A prompt could also be rendered for one engine and launched on another with nothing to catch it.
Now each producer renders its prompt and edit directive for the engine its model choice resolves to,
and `Prepare` refuses a launch whose prompt was rendered for a different engine.

## What changes

- `shuttleengine.Spec` gains `PromptEngine`, the engine its `Prompt` was rendered for, and `HandWrittenPrompt`.
- `CheckPromptEngine` runs first in the Claude and Codex `Prepare`.
  It refuses a spec whose `PromptEngine` differs from the engine's name, or is unset.
  The error names both engines and the producer's model setting as the way forward.
- `EngineSet.EngineName` and `Runner.EngineName` (the `EngineNamer` interface) return the engine a model choice resolves to,
  the set's default name for an empty choice.
  Producers pass that name to their renderer and set it on the spec.
- Every producer outside `seatengine` has one prompt render function that takes the engine name and needs no session.
  Where the launching function filled the prompt inline, the render is split out of it.
  This covers `shedadapters` (`judgeCall`, `runSeedSpawn`), `websterengine`, `treadleengine`, `samplerengine`,
  `landingshed`, `frictionengine`, `mergeresolve`, `sheddriver`, `gatefixer`, `loomcli`, `dobbyengine`, `shedarbiter` and `weaverengine`.
- `editdirective.Directive` takes the engine; `EngineDirective` is gone.
- The shared `edit-directive` stencil is removed.
  Each engine's block copy now carries every rule of the shared directive:
  `lym loom fmt` for Go, the scratch-file rules, renames, and other-worktree work only when the parent or operator says so.
  The Claude copy is the old shared text, and assist seats now get the full directive.
- `lym shuttle run` marks its spec `HandWrittenPrompt` for inline `--prompt` text or a `--prompt-file`.
  Such a spec is not refused, and its manifest names the prompt hand-written.
- The orch spec names the Claude engine and its prompt engine through the `orchEngine` constant in `internal/orchcli`,
  so a Codex default engine never takes the orch role.

## Design decisions

- The `orchEngine` literal is a recorded exception to the rule that a caller never names an engine, and no orch config key is added.
  The orch relies on Claude-only features (remote control, hooks, the reed RC button),
  so a key whose only valid value is `claude` would be a knob that breaks the orch when turned.
- A scan in `internal/shuttleengine` (`promptengine_enforcement_test.go`) fails on a production `Spec` literal that sets `Prompt`
  but neither `PromptEngine` nor `HandWrittenPrompt`.
  It also fails on `HandWrittenPrompt` set outside `internal/shuttlecli`.
  A literal that sets no `Prompt`, such as an attach probe's, is skipped.
  The scan is syntactic, so it does not see a spec filled field by field.
- `PATTERN-shuttle-provider-seam` and `PATTERN-stencil-ownership` are updated: the engine copies are the edit directive's only statement.

## Behavior to know

- A hub that edited its seeded copy of the shared edit directive loses that edit, since nothing reads the shared stencil now.
- A hub that edited its seeded engine copy keeps it as is, without the rules the shared directive adds,
  because `stencilstore` never overwrites an edited stencil.

## To check by hand

- Producer edit-directive copy untested: a producer-level test that its prompt carries the launch engine's directive copy
  may be missing in some producer.
  Check each producer's render test.
- A real launch of one producer under a Codex default engine, and one `lym shuttle run` with `--prompt` and with `--prompt-file`,
  were not run against live sessions.
