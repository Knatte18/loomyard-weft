# Decision record — VS Code colours by project and worktree, and lym ide seed with no slug

## Goal

A VS Code window shows which project a worktree belongs to on its title bar, the prime's green or its beam's colour, and the worktree itself on its activity bar, with text coloured by the background's luminance and a window title of `<SHORTNAME>: <slug>`.
`lym ide seed` with no slug seeds the worktree the current directory sits in, the prime included.

## Scope

- `internal/vscode`: the palette with the darker green `#1b5e20`, the beam and activity-bar pickers, the luminance foreground, and the settings writer that writes title bar, activity bar and window title.
- `internal/ideengine`: resolving a worktree's project colour and worktree colour from the prime, the board's beam entries and `hubgeom.ParentOf`, and the window title from the hub shortname, for `Spawn` and the seed verb.
- `internal/idecli`: `lym ide seed` with an optional slug; its help names the prime as well as task worktrees.
- `internal/boardengine`: a beam project records its colour, `beam_color`, written by the owner setter in the same write as the beam flag.
- `internal/beamcli`: `lym beam create` asks the owner write to record the beam's colour, picked from the palette less green and the colours other open beams hold.
- `internal/fabricengine`: a beam's `.code-workspace` file splices the beam's own `.vscode/settings.json`, so it carries the beam's title and colours.
- Docs: the package docs of the touched packages and `docs/overview.md`'s Seed-IDE line.

Out of scope: `lym ide open` keeps its required slug, and `lym fabric clone` stays IDE-free.

## Decisions

### D1: The beam's colour is a board field on the beam project

Decision: a beam project carries `beam_color`, set only by `boardengine.SetOwners` when a `Beam` request records the beam and the entry has no colour yet; the request's `BeamColor` function picks it from the colours the board's other open beams hold, inside the board write lock.
Rationale: the board is the one record every machine shares, so the colour stays the same at every open and on every machine; picking under the lock keeps two concurrent `lym beam create` runs from taking one colour.
The field is not an upsert key, so no board write but the owner setter sets it; `normalizeTask` drops it from any entry that is not a beam project.

### D2: `lym beam create` re-runs its owner step for a beam with no colour

Decision: the owner step also runs when the beam is recorded here but carries no `beam_color`, so re-running `lym beam create` on a beam made before this change records one.
Rationale: the step is idempotent and already the only writer of the beam flag; it bounds the new write to the colour of the named beam.

### D3: Colours per worktree kind

Decision: the prime takes green on both bars; a beam takes its recorded colour on both; a task takes its parent's project colour (green under the prime, the beam's colour under a beam) on the title bar and a palette colour other than that on the activity bar, avoiding the activity colours of sibling tasks, worktrees whose title bar holds the same project colour, while one is free, and keeping its own current activity colour when that is still allowed and free.
A beam with no recorded colour reads one picked the same way as `lym beam create` would, without recording it.
Rationale: siblings are found from the settings files already on disk, with no extra geometry read; keeping a valid colour makes re-seeding stable.

### D4: lym owns its keys in an existing settings.json

Decision: the settings writer creates `settings.json` as before, and in an existing one rewrites only the keys lym owns, the title-bar and activity-bar keys under `workbench.colorCustomizations` and `window.title`, keeping every other key; a file that does not decode as JSONC is left untouched with a warning.
The status-bar and border keys are never written.
Rationale: without it the prime and every worktree seeded before keep the old colour and title forever; an operator's other settings survive, though comments in a rewritten file do not.

### D5: Text colour by WCAG relative luminance

Decision: the foreground is black when the background's relative luminance is above 0.179, the point where black and white give equal contrast, and white otherwise; the inactive title-bar and activity-bar text take the same colour at `aa` alpha.
Rationale: one rule for every palette colour, so a lighter palette entry stays legible.

### D6: Window title

Decision: `window.title` is the hub shortname upper-cased, `: ` and the worktree's slug; a hub with no recorded shortname takes the slug alone.
Rationale: the shortname is the hub's display name everywhere else; a missing record must not fail a seed.

### D7: `lym ide seed` with no slug

Decision: with no argument, `lym ide seed` seeds the worktree the resolved location names, `Location.WorktreeName`, prime included; with a slug it behaves as today, and the prime's name is accepted as a slug as well.
Rationale: the location is already resolved by the verb's pre-run, so no new geometry read is needed.

### D8: The beam's workspace file splices the beam's settings

Decision: `RegenerateHubWorkspace` splices into each beam's `.code-workspace` file the beam's own `.vscode/settings.json` when it exists, and the prime's settings otherwise as today.
Rationale: workspace settings override folder settings, so the beam file would otherwise show the prime's green and title.

## Constraints

- `PATTERN-pair-parent`: a task's parent is read only through `hubgeom.ParentOf`.
- `PATTERN-owner-machine`: the beam colour is written only inside `boardengine`'s owner setter.
- `PATTERN-tolerant-records`: the new board field round-trips and unknown keys stay kept.
- `PATTERN-test-speed`: tests are tagged at the lowest fitting tier and parallel by default.

## Auto-mode assumptions

- A darn run holds no interview; every call above is the writer's own self-pick.
- An operator's own colour in a seeded worktree's settings is replaced at the next seed or spawn; the design gives lym the colour of every worktree.

## Open risks

- Rewriting an existing `settings.json` drops its comments; files lym wrote carry none.
- Board files written by an older binary keep `beam_color` as an unknown key and round-trip it, so a mixed fleet keeps the value.

## Acceptance criteria

- The prime's settings carry `#1b5e20` on the title bar and activity bar, and title `LY: <prime>` for shortname `ly`.
- A task under the prime carries green on the title bar and a non-green colour on the activity bar that no sibling task holds while one is free.
- A task under a beam carries the beam's colour on the title bar and a different colour on the activity bar.
- A beam carries its recorded colour on both bars; `lym beam create` records a colour other than green and other than any other open beam's.
- Foreground is white on every dark palette colour and black on a light background.
- An existing settings.json keeps an unrelated key and gets the new colour keys; an undecodable one is left byte-identical.
- `lym ide seed` with no slug seeds the current worktree, the prime included.
- A beam's workspace file carries the beam's settings when the beam has them.
- The board keeps `beam_color` through load, save and replay, and drops it from an entry that is not a beam project.
