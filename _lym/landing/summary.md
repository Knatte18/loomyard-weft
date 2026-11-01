# Heddle ships lym's own layer; batching moves into CLAUDE.local.md

## What changes

Heddle now reads two layers of instruction entries instead of one.

- **Shipped layer.** `internal/heddleengine/shipped.json` is embedded in the binary, uses `heddle.json`'s entry format, and is read-only: it changes only with a lym release.
  It is decoded once at package load, and a build whose shipped entries fail `ValidateEntry` or repeat a slug panics.
- **Repository layer.** `heddle.json` in the Bolt checkout, unchanged.

Render, `Show` and `Passages` read both layers in one order through `Layers.Ordered`: the shipped entries first, then the repository's.
A file target writes the shipped texts under `# LoomYard's own instructions`.
When shipped texts come first, the repository's texts follow under `# This repository's instructions`.
A target with no shipped entry, such as `codex-file` today, renders exactly as before.

The batching sentence ("Request every call whose arguments you already know in one message, …") is the shipped layer's first entry (`lym-batching`, target `claude-file`).
It no longer appears in claudeengine's `append-system-prompt.txt`, so a session gets it once, in the rendered `CLAUDE.local.md`.
Claude Code presents CLAUDE files as overriding its defaults.
The trial asks whether that framing moves agents more than the appended system prompt did.

`lym heddle`:

- `set` and `remove` refuse a shipped slug, naming `--layer lym` as the way to tune it.
- `list` prints each entry's `layer` (`shipped` or `repository`) and marks an `overridden` shipped entry.
- `get` finds shipped entries too and prints the text renders use now.

### Hub overrides of shipped entries

A new hub-wide config module, `heddle.yaml`, holds one key, `lym_override`, which is off by default.

- `lym heddle set <slug> --layer lym --text-file <f>` stores an override of a shipped entry's text in `heddle-lym-override.json`, beside `heddle.json`.
  It is refused while the setting is off, with `lym config heddle --set lym_override=true` as the way forward.
  It is also refused for a slug lym does not ship, for a missing `--text-file`, and when `--order` or `--targets` is given.
- `lym heddle remove <slug> --layer lym` drops an override whatever the setting holds.
- With the setting on, render, show, passages and `list` use the override's text in place of the shipped text.
  With it off, every override is ignored.

## Design decisions

- **Where the channel lives.** The parent's decisions in the decision record settled this.
  The run first tried a `CLAUDE.md` in the run directory loaded through `--add-dir`, then a code-supplied section in heddle's renderer, before the shipped layer.
  Only the shipped layer is in this change: it reuses heddle's targets and rendering, and keeps lym's text in lym's code per `PATTERN-lym-instructions-in-code`.
- **`heddle.yaml` loads degrading.** It loads through `configengine.LoadOrTemplate`, and `internal/heddleengine` joins the pinned degrading set in `cmd/lym/configstrictness_test.go` and `pattern/PATTERN-config-layers.md`.
  Renders run at worktree creation and at every session launch.
  A hub that has not reconciled since this binary landed has no `heddle.yaml` yet, and a strict load would fail every render and passage read there.
  `desktophook` takes the same choice for the same reason.
- **The override file has no replayer.** Heddle's Bolt replayer still claims only `heddle.json`.
  Fabric's Bolt replay cherry-picks a path no replayer claims, which is enough for a tuning file that one hub, loomyard, writes.
- **The key is `CategoryCrossPair`.** It changes what every worktree of the hub renders, so the local layer refuses it.
- **Help text trimmed.** Adding `heddle` to the hub-wide module list in `lym config`'s help pushed it 7 bytes over the 2000-byte cap.
  "the JSON success output's" became "the JSON output's".

## Effects to know

- Every worktree now gets a rendered `CLAUDE.local.md` holding the shipped section, even in a hub with no claude-file entry.
  An existing worktree takes it at its next take-in render or `lym heddle render`; a session takes it at its launch or its next compaction.
- An operator's own unmarked `CLAUDE.local.md` is still left untouched.
  It now draws the render's warning in every hub, not only in hubs with claude-file entries.
- The registry gains the `heddle` module, so the registry fingerprint changes and the next start verb's hub reconcile runs.

## Follow-ups

- Loomyard's own hub turns the setting on with `lym config heddle --set lym_override=true` once this binary is in use.
- `heddle-orch-target`, in review, also changes `internal/heddleengine` (`render.go`, `target.go`), and the later of the two to land merges its target in.
- `plan-edits-before-gate` adds its instruction as the shipped layer's second entry in `shipped.json`.
- Note `agent-turns-effect` measures the effect.

## Tests

- `internal/heddleengine`: the render tests pin the shipped section and the repository's heading, a `CLAUDE.local.md` with no entry, and an override used only while the setting is on.
  The store tests cover set and remove refusing a shipped slug, and `override_test.go` covers override set, replace, remove and their refusals.
- `internal/heddlecli`: the tier1 `OverrideVerbs` test drives every `--layer lym` refusal and its way forward, the override write and its sync, and `list` and `get` with the setting on and off.
  `EntryVerbs` expects the shipped slugs first.
- `internal/shuttleengine/claudeengine`: the appended-prompt tests now expect the deny notice and creel sentence alone.
- `internal/configreg`: `heddle` is pinned as hub-wide.
- The scoped tier0 gate and tier1 over `heddlecli`, `heddleengine`, `configreg`, `configcli`, `hubreconcile` and `configsync` pass.
