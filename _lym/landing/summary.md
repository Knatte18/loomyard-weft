# Heddle: a shipped layer carries lym's batching instruction

The batching sentence every shuttle-launched Claude session gets moves from the appended system prompt (`batchingSentence` in `claudeengine`) into the rendered `CLAUDE.local.md`.
Claude Code reads a project's CLAUDE files under "These instructions OVERRIDE any default behavior", a stronger frame than a sentence appended to its long system prompt.
The change tests whether that frame raises calls per turn above the 1.7 measured on Webster forks after `turn-batching`.

Heddle gains a second, read-only layer to carry it, so lym's own instructions stay in lym's code while sharing heddle's targets and rendering.
The same layer is where `plan-edits-before-gate` adds its instruction.

## What changes

- `internal/heddleengine` embeds a shipped layer, `shipped.json`, in the format of `heddle.json`.
  Its first entry, `lym-batching`, targets `claude-file` and holds the former sentence word for word.
  The layer is validated at package load and panics on a malformed or duplicate entry.
- Rendering, `lym heddle show` and the standing passages read the shipped layer and the repository's `heddle.json` together for every target.
  Shipped entries render first under `# LoomYard's own instructions`, the repository's after under `# This repository's instructions`.
- `claudeengine` no longer appends the batching sentence, so a session never gets it twice.
  The standing passages now come from heddle's layers.
- `lym heddle list` marks each entry's layer.
  `set` and `remove` refuse a shipped slug, and `set` refuses a new repository entry under one.
- A hub setting, `heddle.lym_override` (new `heddle.yaml` module, hub-wide, off by default), enables tuning without a build.
  With it on, `lym heddle set <slug> --layer lym` stores a replacement text in `heddle-lym-override.json` beside `heddle.json`, and `remove --layer lym` drops it.
  Rendering, `show` and passages then use an overridden entry's text; the entry keeps the shipped order and targets, and `list` marks it.
  With it off, overrides are ignored and `set --layer lym` is refused.
- `PATTERN.md`, `PATTERN-heddle-instructions.md`, `PATTERN-config-layers.md` and `docs/overview.md` describe the second layer.

## Design decisions

- The shipped layer is changed only by a lym release; the override exists because tuning and benchmarking lym's text in loomyard needs a faster loop than a new binary.
  Once a text proves itself, a later task copies it into `shipped.json` and removes the override.
- A broken override file does not stop reads.
  The overrides are dropped, the shipped texts stand, and a warning is carried (`Layers.OverrideWarning`).
  The setting only tunes lym's text and must not block the repository's own entries.
- Overrides replace only the text; order and targets stay the shipped entry's.
- `lym_override` is hub-wide and the local config layer refuses it, since it changes what every worktree renders.
- Removing the sentence from the append prompt and adding it to the shipped layer are in the same change, so there is no state in which a session gets both or neither.

## To check

- Run two or three tapestry runs and measure calls per turn with `tools/forkaudit` against 1.7. Not run here. The effect is also measured for the whole project by note `agent-turns-effect`.
- A malformed override file makes every read fail rather than warn: `bad-override-file-fails-all-reads`. Check the `ReadLayers` path against the warning behavior above, and the `show` and `passages` callers.
- A repository entry whose slug collides with a shipped slug (for instance one created before the shipped entry existed) stays open to change and removal but cannot be recreated once removed: `shipped-slug-collision-stuck`. Check that the way-forward text in `refuseShipped` gets such a repository out.
- `lym heddle show` and passages with an override in force have no direct test: `show-passages-override-untested`. Check by hand with `heddle.lym_override` on and an override set.
- Confirm a freshly rendered `CLAUDE.local.md` shows the two sections in order, and that a launched session's append-system-prompt file no longer holds the batching sentence.
- `heddle-orch-target`, in review, also changes `internal/heddleengine`; its target needs taking in when it lands.
