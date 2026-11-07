# Heddle renders lym's Claude settings into .claude/settings.local.json

Every Claude session in a lym worktree now gets the settings lym needs, one the operator starts by hand included.
Before, they came only from the `--settings` document lym passes its own launches, and from a checked-in `.claude/settings.json` that only this repository had.

## What changes

- **New heddle file target `claude-settings`** (`internal/heddleengine/target.go`, `settings.go`).
  It renders into the anchor's `.claude/settings.local.json`.
  Each entry's text is one JSON object.
  - A render folds the entries in render order, shipped first:
    objects merge key by key at every depth, a later scalar replaces an earlier one, and lists join without repeats.
  - The folded object merges into the file by the same rules.
    Keys the file holds that heddle never set stay, such as a permission Claude Code wrote on "don't ask again".
    A scalar heddle sets replaces the file's.
  - Each render records the folded object at `<scratch>/heddle/claude-settings.rendered.json`, beside the render lock.
    The next render first takes out of the file what that record holds and no entry sets now: keys, list items, and containers that removal empties.
    A scalar the file changed since is kept.
  - With no entry left, a file the removal empties is deleted.
  - A file holding no JSON object is left alone with a warning naming the way forward.
  - The file can carry no marker, so the marker check applies to the text targets only.
  - `lym heddle show claude-settings` prints the folded object.
- **Validation.**
  `ValidateEntry` refuses an entry aimed at `claude-settings` whose text is no JSON object, or that also names a text target.
  It runs at `lym heddle set`, in the Bolt replayer and when the shipped layer decodes.
  `SetOverride` applies the same check to an override of a shipped settings entry.
  An unparseable text in a hand-edited file is left out of the fold with a warning.
- **Shipped entries** (`shipped.json`):
  - `lym-no-attribution`: `attribution` with empty `commit` and `pr`.
  - `lym-plugins`: `ly@loomyard` and `scribe@scribe` on, with `scribe`'s marketplace under `extraKnownMarketplaces` (GitHub `Knatte18/scribe`).
    `ly@loomyard` gets no marketplace, because it installs together with the `lym` binary.
  - `lym-tool-deny`: the `permissions.deny` list from the repository's `.claude/settings.json`.
- **`.claude/settings.json` is removed.**
  Its `mill@millhouse: false` is this machine's setting and already lives in `~/.claude/settings.json`.
- **Fabric.**
  `warp_exclude` in `internal/fabricengine/template.yaml` gains `.claude/settings.local.json`, so the file stays untracked.
  A hub whose `fabric.yaml` sets its own list gains the path through a new config reinterpretation on the fabric module (`heddle-claude-settings-excluded`).
  Its rewrite is `fabricengine.AppendExcludePaths`.
  It appends missing paths, keeps comments and other keys, and leaves a file that does not set the key unchanged.
  The fabric seam already seeds the exclude lines before it fires, so heddle writes only once the line is in place.
- `writeRendered` stages under the target's name, so a target file in a subdirectory stages directly in `.heddle`.
  It also creates the target file's directory.

## Design decisions

- **Targets do not declare their paths for fabric to read.**
  The record asked whether they should.
  Fabric may learn nothing of heddle (`PATTERN-fabric-boundary`), and `fabricengine` cannot import `heddleengine`, which imports it.
  `warp_exclude` also stays the hub's own editable config.
  So a new target is still a second edit in fabric's template.
  The reconcile rewrite is generic, and `configreg` wires it with `heddleengine.ClaudeSettingsFile`, so fabric's Go code names no heddle file.
- **The record lives in the scratch directory, not in the anchor's `.heddle`.**
  `.heddle` is the staging directory that each render empties and that blocks any non-`.tmp` content.
  The scratch directory's heddle directory is per worktree and never tracked.
- **The `--settings` document still sets `attribution`.**
  A session in a worktree not yet rendered, or on an older binary's worktree, keeps no co-author.
  Per-session and per-role settings stay there, as the record asks.
- **No model or effort defaults.**
  The record leaves them to `agent-skills-and-effort`, which can render them through this target.

## For the reviewer to check

- **Existing worktrees get the file only at their next `lym heddle render`.**
  Only a worktree added or cloned after this change gets it at once, and a take-in re-renders only the text targets.
  This repository's checked-in `.claude/settings.json` goes away when this lands.
  Until `lym heddle render` runs in each existing worktree, sessions there run without `ly@loomyard` on and without the deny list.
- **Not checked in a live launch:** whether Claude Code offers to install a marketplace and plugin declared in `.claude/settings.local.json`, as it does for one in a project's `.claude/settings.json`.
  The record asks for this check, and it needs an interactive Claude Code launch.
- **Not found in the code:** the hub-start notice that names which plugin to install.
  The record describes it as existing behaviour, and this change does not add it.
