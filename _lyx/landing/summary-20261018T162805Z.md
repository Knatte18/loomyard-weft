# Named run profiles in profiles.yaml, picked per task by a profile field

A hub keeps named sets of run settings in one file on Bolt, `profiles.yaml`.
A board task picks one by naming it in its new `profile` field, so a task that wants other review rounds than the hub's no longer carries the numbers in `run_settings`.
Batten's Seed-Child writes the profile's settings into the pair's local config with the task's own `run_settings` laid on top.

## What changes

- New package `internal/runprofile` holds the `profiles` config module: the template, the layered-load spec, `Load` and `Profiles.Lookup`.
  The template seeds the starting profiles from the board entry, `light`, `budget` and `docs`.
  `Load` decodes the stacked file without filling it from the template, so a starting profile the operator removed stays removed.
  With no file present it reads the template.
- `configreg` registers `profiles` as hub-wide and seed-only.
  The hub reconcile seeds an absent file and never rewrites a present one.
  The module's load check runs every profile through `PrepareRunSettings`, so a profile is held to the rules of a task's `run_settings`, and a refusal names the profile.
- New `configreg.LayerRunSettings` prepares a profile and a task's settings, lays each module's task document over the profile's through `yamlengine.Merge` with the module's open maps, and checks the merged document again.
- New `configengine.Spec.JSONEntries` flag, set only by `profiles`, makes `configengine.Set` write each pair as one whole top-level entry from a JSON object through new `yamlengine.SetEntries`.
  `lyx config profiles --set <name>='<json object>'` then adds a profile or replaces one whole, with the other profiles and comments left in place.
  The written file still passes strict validation and the module's load check, or it is put back.
  A rename is adding the new name and deleting the old block in the editor, `lyx config profiles`.
- `boardengine.Task` gains `Profile` (`profile`, omitted when empty), an upsert payload key, and `""` clears it.
  The run lock does not treat it as scope, as it does not treat `run_settings` as scope.
  The README entry cell shows it as a `profile: <name>` bullet before the run settings.
- `boardcli`'s run-settings check also checks `profile` on `upsert`, `upsert-batch` and `merge`.
  It must be a string, and a non-empty one must name a profile in the hub's file.
  An unknown name is refused naming the known profiles, with `lyx config profiles --set` as the way forward.
  A profile change gets the same task-session refusal and running-pair notice as a `run_settings` change.
- New verb `lyx board profiles` prints `{"profiles":[{"name":…,"settings":{…}}]}` in name order, `[]` when the file holds none.
- battenshed's Seed-Child seam carries the profile: `ReadRunSettings` returns a new `battenshed.RunSettings{Profile, Settings}`, and `PrepareRunSettings` takes it.
  battencli's new `prepareRunSettings` loads the profiles from the board dir and calls `LayerRunSettings`.
  An unknown profile, a profiles file that fails to decode, or a refused key wraps `ErrInvalidRunSettings`, so the row stands Stuck.
  The Stuck reason's way forward now names the profile field and `lyx config profiles` beside `run_settings`.
- Docs: the new package's `doc.go`, the `boardengine`, `boardcli`, `battenshed`, `configcli`, `configengine` and `yamlengine` package docs, `docs/overview.md` and the `ly:board` skill.

## Design decisions

- The template's profiles are classified cross-pair in the spec, so the registry test's every-unit-classified rule holds.
  No profile is open to the local layer.
  The categories come from the template's own keys rather than a list, so changing a starting profile needs no second edit.
- A hand-broken `profiles.yaml` stands Seed-Child Stuck with the way forward rather than failing hard, since the operator fixes it the same way as an unknown profile.
  An unreadable file stays a hard error.
- `SetEntries` writes JSON scalars in plain YAML style, quoted only where the text needs it, so a profile set from JSON reads like a hand-written one.
- A task session can no longer change its own `profile`, just as it cannot change its own `run_settings`: a profile lowers review rounds the same way.
  The guard is the existing slug guard: it stops a mistaken write, not a determined one.
- The help pages of `lyx board upsert` and `lyx config` grew past the 2000-byte cap once the new field and module were added.
  A few lines of each were shortened, dropping asides such as `body`'s "(proposal / background)" and `--body-file`'s "every other field still comes from the payload".

## Not changed

- The hub default of about 3-3-2 review rounds is untouched, and is not a profile.
- A hub whose reconcile has not yet run after this binary reads the template's profiles until it does.
