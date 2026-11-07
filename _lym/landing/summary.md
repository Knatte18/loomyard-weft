# Heddle renders .claude/settings.local.json in every hub worktree

Every worktree of a hub now gets a `.claude/settings.local.json` that heddle renders, so a Claude session the operator starts by hand gets the same settings lym's own sessions get through `--settings`.
The file holds what lym needs in any session of a hub it initialized:

- `attribution` with empty commit and pull-request texts, so no Claude co-author or session link lands in commits and pull requests.
- `enabledPlugins` with `ly@loomyard` and `scribe@scribe` on, and `extraKnownMarketplaces` declaring `scribe` (GitHub `Knatte18/scribe`).
  `ly@loomyard` is switched on with no marketplace declared: lym's install delivers it together with the `lym` binary.
- `permissions.deny` for the tools lym never uses, which cuts a role's first request in tokens (see `docs/research/tokens/role-tool-schemas-2026-11-07/report.md`).

The repository's checked-in `.claude/settings.json` is removed, since these entries replace everything in it but `mill@millhouse: false`.
That line is this machine's, and `~/.claude/settings.json` already sets it.

## Design decisions

- **New target kind.**
  `claude-settings` in `internal/heddleengine/target.go` is a JSON-object file target.
  Its entries each hold one JSON object, and an entry aimed at it must name no text target (`lym heddle set … --targets claude-settings`).
  A text that is not one JSON object is refused when written.
  The three shipped entries (`lym-no-attribution`, `lym-plugins`, `lym-tool-deny`) are in `shipped.json`.
- **Fold and merge.**
  `internal/heddleengine/settings.go` folds the entries in order, shipped first: objects merge key by key at every depth, a later scalar replaces an earlier one, and lists join without repeats.
  The folded object then merges into the file by the same rules, except that a heddle scalar overwrites the file's.
  Every other key in the file stays, such as a permission Claude Code wrote on "don't ask again".
- **Removal.**
  Heddle records what it set at each render in a `.rendered.json` record beside the render lock.
  A key or list item it set at the last render that no entry sets any more is removed at the next.
- **Untracked file.**
  `warp_exclude` in `internal/fabricengine/template.yaml` gains `.claude/settings.local.json`, and fabric writes the list before the worktree seam fires.
  A hub whose `fabric.yaml` sets its own `warp_exclude` gets the path through a config reconcile reinterpretation (`heddle-claude-settings-excluded`, built on the new `fabricengine.AppendExcludePaths`).
  The `warp_exclude` key name is now the constant `fabricengine.ExcludeListKey`.
- **Left out.**
  Model and effort defaults are not rendered here; they belong to `agent-skills-and-effort`, which can add them as further entries to this target.
  What differs per session or role (hooks with a run's paths, session scope, creel's hooks) stays in the `--settings` document.
- Docs updated: `docs/overview.md`, `pattern/PATTERN-heddle-instructions.md`, the `heddleengine` and `fabricengine` package docs and the `lym heddle set` help.

## To check by hand

- In a launch, check that Claude Code offers to install the `scribe` marketplace and plugin declared in `settings.local.json`, as it does for one declared in a project's settings.
  This was not run.
- `internal/heddleengine/settings_test.go` may not cover every render case that keeps a stale key on an invalid entry, or that removes a key no entry sets any more.
  Check that the keep and take-out rules are covered there.
