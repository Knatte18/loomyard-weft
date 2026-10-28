# Colour VS Code by project and worktree; lym ide seed takes no slug

A worktree's VS Code window now shows its project on the title bar and the worktree itself on the activity bar.
`lym ide seed` with no slug seeds the worktree the current directory sits in, the prime included.
Before, every worktree got one title-bar colour, nothing showed which beam it belonged to, and seeding a fresh prime meant spelling out its name.

## What changes

- **Colours.**
  Only the title bar (active and inactive, the same colour) and the activity bar are written, since they are the only areas that keep their colour in an unfocused window.
  - The prime takes green on both bars.
    The palette's green, `mainColor` and `palette[0]` in `internal/vscode/color.go`, is now the darker `#1b5e20`.
  - A beam takes its own colour on both bars.
  - A task takes its parent's project colour on the title bar (green under the prime, the beam's colour under a beam).
    Its activity bar takes a palette colour other than that, avoiding its sibling tasks' activity colours while one is free and keeping its own current colour when that is still allowed and free.
  - Text colour follows WCAG relative luminance: black above 0.179, white otherwise.
    The inactive title-bar and activity-bar text take the same colour at `aa` alpha.
- **Window title.**
  `window.title` is the hub shortname upper-cased, `: ` and the worktree's slug, such as `LY: loomyard`.
  A hub with no recorded shortname takes the slug alone.
- **`lym ide seed [<slug>]`.**
  With no slug it seeds the worktree `Location.WorktreeName` names; with a slug it behaves as before, and the prime's name is accepted as one.
  The help names the prime as well as task worktrees.
  `lym ide open` keeps its required slug and `lym fabric clone` stays IDE-free.
- **Beam colour.**
  A beam project carries a new board field, `beam_color`.
  `lym beam create` asks the owner write to record it, picked from the palette less green and the colours other open beams hold.
  Re-running `lym beam create` on a beam recorded here with no colour records one.
- **Beam workspace file.**
  `RegenerateHubWorkspace` splices each beam's own `.vscode/settings.json` into its `.code-workspace` file when it exists, and the prime's otherwise, so the beam's file no longer overrides it with the prime's green and title.
  An unreadable beam settings file falls back to the prime's for that beam and leaves the other beams' files and the stale-file removal running.

## Design decisions

- **`beam_color` lives on the board, set only by `boardengine.SetOwners`.**
  The board is the one record every machine shares, so the colour is the same at every open.
  The request's `BeamColor` function picks it inside the board write lock, so two concurrent `lym beam create` runs cannot take one colour.
  It is not an upsert key: `normalizeTask` drops it from any entry that is not a beam project, and the `lym board upsert` help and the `UpsertPayloadKeys` comment name it as output-only.
  Board files written by an older binary keep it as an unknown key and round-trip it.
- **A beam with no recorded colour reads one picked as `lym beam create` would, without recording it.**
  The colour can change once, when it is recorded.
- **Siblings come from the settings files on disk**, with no extra geometry read.
  A task's parent is read only through `hubgeom.ParentOf`.
  An unreadable board or a failing `ParentOf` reads the worktree as a task under the prime, with a warning.
- **lym owns its keys in an existing `settings.json`.**
  The writer creates the file as before; in an existing one it rewrites only the title-bar and activity-bar keys under `workbench.colorCustomizations` and `window.title`, keeping every other key.
  A file that does not decode as JSONC is left byte-identical, with a warning.
  Comments in a rewritten file are lost; files lym wrote carry none.
  Without this, the prime and every earlier-seeded worktree would keep the old colour and title.
- **An operator's own colour in these keys is replaced at the next seed or spawn**, since lym assigns every worktree's colour.
- The status-bar and border keys are never written, as the status bar loses its background and the active border is not drawn in an unfocused window.

Docs updated in the touched packages' `doc.go`, `docs/overview.md`'s Seed-IDE line and the `seed_ide_command` comment in `internal/battenshed/template.yaml`.

## To check by hand

Not run: a real VS Code window.
Seed a prime, a beam and tasks under each, open them, and look at the title bar, activity bar, text colour and window title, focused and unfocused.
Also run `lym ide seed` with no slug from the prime and from a task.

The review of this change ended without converging; check each of these findings by hand:

- `internal/fabricengine/doc.go` (the hub workspace section): the doc must say a beam's file splices the beam's own settings, with the prime-only sentence limited to the hub file.
- `internal/boardengine/replay.go:57` and `task.go:279`: `beam_color` must survive a replayed commit and be dropped from a non-beam entry, each pinned by a test.
- `internal/fabricengine/hubworkspace.go:135-142`: one beam with unreadable settings must not abort the regeneration of the others.
- `internal/boardengine/store.go:456-458`: `beam_color` in `lym board get` output must be documented as output-only for `lym board upsert`, with a key test.
- `internal/ideengine/spawn.go` (`lookFor`): the unreadable-board and failing-`ParentOf` fallbacks need tests.
- `internal/beamcli/create.go:164`: a test must pin that `lym beam create` passes the colour picker.
- `internal/beamcli/doc.go:22`: create step 2 must mention the colour and the re-run for a beam with none.
