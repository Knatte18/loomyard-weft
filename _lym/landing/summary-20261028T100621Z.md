# VS Code colours by project and worktree, and lym ide seed with no slug

A worktree's VS Code window now shows two things: its project on the title bar and the worktree itself on the activity bar.
These are the only two areas that keep their colour when the window loses focus.

## What changes

- **Colours.**
  The prime takes green on both bars, and the green is now the darker `#1b5e20`.
  A task under the prime takes green on the title bar and some other palette colour on the activity bar.
  A beam takes its own colour on both bars.
  A task under a beam takes the beam's colour on the title bar and a different colour on the activity bar.
  A task's activity-bar colour avoids its siblings' colours while the palette has one free.
  Siblings are the worktrees whose title bar holds the same project colour.
  A task keeps its current colour as long as that colour is still allowed and free, so seeding again does not change it.
  Text on either bar is black or white, chosen by the background's WCAG relative luminance.
  No status-bar, border or tab colour is written.
- **Window title.**
  `window.title` is the hub shortname in upper case, a colon and the worktree's slug, for example `LY: loomyard`.
  A hub with no recorded shortname shows the slug alone.
- **Beam colour on the board.**
  A beam project carries a new `beam_color` field.
  `boardengine.SetOwners` writes it in the same commit as the beam flag, using the new `OwnerRequest.BeamColor` picker.
  The picker chooses from the colours the other open beams hold, and it runs inside the board write lock, so two concurrent creates cannot take the same colour.
  `lym beam create` passes `vscode.PickBeamColor` as the picker.
  It also runs its owner step again for a beam that is already recorded here but has no colour.
  The replay merges the field.
  Only a beam project keeps it, and no upsert key sets it.
- **settings.json.**
  `vscode.WriteConfig` now takes a `Look` (title, project colour, worktree colour).
  In an existing `settings.json` it sets only the keys lym owns: `window.title` and the title-bar and activity-bar keys of `workbench.colorCustomizations`.
  It keeps every other key.
  A file that does not decode as a JSONC object is left untouched, with a warning.
- **Beam workspace file.**
  A beam's `<RepoName>-<beam>.code-workspace` now splices the beam's own `.vscode/settings.json`, so it carries the beam's title and colours.
  When the beam has no settings file, it falls back to the prime's settings as before.
- **`lym ide seed [<slug>]`.**
  The slug is optional: with none, the command seeds the worktree it runs in, the prime included.
  `ideengine.SeedTask` becomes `Seed`.
  `lym ide open` still requires a slug.

## Design decisions

- The beam colour lives on the board rather than in a file on the machine, so every machine shows the same colour for a beam.
- `lym beam create` picks a beam's colour only once. Until a beam made before this change gets one, the IDE picks a colour for it each time it opens and records none.
- Rewriting lym's own keys in an existing `settings.json` replaces the old write-once rule. Without it, the prime and every worktree seeded earlier would keep the old green and title until the operator deleted the file.
  The cost is that a rewritten file loses its comments, and an operator's own title-bar colour is replaced at the next seed.
- The prime-name log-level split (`primeResolveLogLevel` and the `spawnTarget` enum) is removed.
  When the prime's name cannot be resolved, every worktree's colours are affected, not just a task's, so the failure now always logs at Warn.

## Tests

- `internal/vscode`: tests for both pickers, the sibling scan, `Foreground`, and the full set of colour keys in a new and in an existing `settings.json`.
- `internal/ideengine`: `look_test.go` covers `lookFor` over a hub of plain directories, for the prime, a beam with a colour, a beam without one, tasks under each, and the title with and without a shortname.
- The ide spawn integration checks are updated for the rewritten settings.
- `internal/boardengine`: tests that the owner write records the colour, passes the other open beams' colours to the picker and keeps an existing colour; that load drops `beam_color` from an entry that is not a beam; and that `ParseOwners` reads the field.
- `internal/beamcli`: a test that a beam with no colour runs the owner step again.
- `internal/fabricengine`: a test that a beam's workspace file uses the beam's own settings, and the prime's when the beam has none.
- `internal/idecli`: a test that `lym ide seed` with no slug run in the prime seeds the prime.
