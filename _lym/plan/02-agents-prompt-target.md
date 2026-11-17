# Card 2 — agents-prompt-target

**Create:**
- `plan:internal/heddleengine#agentsPromptTarget` -> `const agentsPromptTarget = "agents-prompt"`

**Edit:**
- `internal/heddleengine#targetTable`
- `internal/heddleengine#promptTargetsFor`
- `internal/heddleengine#passagesOf`
- `internal/heddleengine#reachesFileTarget`
- `internal/heddleengine#Renderer.Show`
- `internal/heddleengine#overviewIntro`
- `internal/heddleengine#TestRender`
- `internal/heddleengine/doc.go`
- `internal/heddlecli#heddleCLI.showCmd`
- `internal/heddlecli#heddleCLI.setCmd`

**Extends:**
- `internal/heddlecli/doc.go`

**Uses:**
- `internal/heddleengine#newRenderFixture`
- `internal/heddleengine#renderFixture.anchorHas`

**Intent:**
Add the shared prompt target `agents-prompt` (D7).
It is a `targetTable` row with `Delivery: DeliveryPrompt` and `RendersInto: [claude-prompt, codex-prompt]`, and no engine or role of its own,
so an entry aimed at it reaches every session `claude-prompt` or `codex-prompt` reaches, the Claude orch included.

- `promptTargetsFor` skips a shared row, as `Render` skips a shared file row, so the shared row reaches a session only through a target it renders into.
- `passagesOf` takes an entry when it names one of the given targets or a shared target that renders into one of them.
  It uses `reachesFileTarget`, whose rule already covers a shared prompt target; its doc comment says it serves file and prompt targets alike.
  So `Passages` and `Show(claude-prompt)` give an `agents-prompt` entry's text, once, in render order, even when the entry also names `claude-prompt` or `orch-prompt`.
- `Show` already refuses a target with `RendersInto`; its message changes from "renders into the files of" to "renders into the targets",
  so it reads true for `agents-prompt`, and keeps the way forward `run lym heddle show <target> with one of them`.
  The input that reaches it is `lym heddle show agents-prompt`, which `showCmd` passes to `Show` unchanged.
- `HEDDLE.md` places an `agents-prompt` entry under the `claude-prompt` and `codex-prompt` sections with no change to `overviewPlaces`:
  it skips every shared row and places an entry through `reachesFileTarget`.
  So that `HEDDLE.md` states the new target even while no entry uses it (`PATTERN-heddle-instructions`),
  `overviewIntro` gains a line naming the two shared targets: `agents-file` lands in both agent files, and `agents-prompt` in every Claude and Codex session's prompt.
- `ValidateEntry` accepts the new name through `LookupTarget`.
  Bound: a machine on an older lym refuses a store holding an `agents-prompt` entry until it upgrades; no shipped entry uses the target.

`heddlecli`'s `set` help adds that `agents-prompt` reaches both engines' sessions,
and its `show` help says `agents-prompt` is refused like `agents-file`, naming the targets to show instead.

Tests, in the existing `TestRender`:
- Its `ShowAndPassagesWriteNothing` subtest gains an `agents-prompt` entry, and an entry aimed at both `agents-prompt` and `orch-prompt`.
  It then expects `Passages(claude, "")` and `Passages(codex, "")` both to hold the first entry's text, `Show(claude-prompt)` to hold it,
  `Passages(claude, "orch")` to hold each entry's text once,
  and `Show(agents-prompt)` to be refused with an error naming `claude-prompt, codex-prompt` and `lym heddle show <target>`.
  Its `Passages(codex)` assertion changes from none to those texts.
- No CLI row is added: `showCmd` passes `Show`'s error through, so the engine row reaches the refusal.

`heddleengine/doc.go`'s prompt-target paragraph, which names `claude-prompt` and `orch-prompt`, is rewritten to name `agents-prompt` and what it reaches.

**ImpactSummary:** one new table row read by every prompt-target lookup in heddleengine; `Passages` gains entries only for stores naming the new target; the `Show` refusal text has one asserting row, updated here; `HEDDLE.md`'s intro gains one line.
