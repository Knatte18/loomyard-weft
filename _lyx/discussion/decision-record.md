# Decision record — run-profiles

## Goal

A hub holds named run profiles in one file, `profiles.yaml` on Bolt, and a board task picks one by its own `profile` field, so a task wanting other review rounds than the hub's names a profile instead of carrying the numbers in `run_settings`.

## Scope

- A new hub-wide, seed-only config module `profiles`, whose file maps a profile name to settings in the shape of a task's `run_settings`, seeded with the starting profiles `light`, `budget` and `docs`.
- A new package `internal/runprofile` holding the module's template, spec and loader.
- `lyx config profiles` opens the file in the editor through the ordinary module path, and `--set <name>='<json>'` adds or replaces one whole profile.
- A `profile` field on a board task, beside `recipe`, checked at every upsert path and refused when it names no profile.
- Batten's Seed-Child writes the named profile's settings into the pair's local layers with the task's own `run_settings` layered on top, refusing an unknown profile.
- `lyx board profiles` prints the profile names and their settings.
- The README entry cell shows a task's profile beside its run settings.
- Docs: the new package's `doc.go`, and the doc comments of `boardengine`, `boardcli`, `battenshed` and `configcli`.

## Decisions

### Profiles are a hub-wide, seed-only, whole-entry config module

Decision: `profiles` registers in `configreg` with `HubWide`, `SeedOnly` and a spec with `WholeEntries`, its template holding the starting profiles.
Rationale: hub-wide puts its one file on Bolt and gives `lyx config profiles` the board write path, guard and commit for free;
seed-only means a reconcile seeds an absent file and never rewrites a present one, so a removed or renamed profile stays as the operator left it;
whole entries make each profile one unit.

### The profile loader reads the stacked file without template fill

Decision: `runprofile.Load` reads the layers through `configengine.StackLayers` and decodes the merged document, so a profile removed from the hub file is never filled back from the template.
With no layer file present it decodes the template.
Each profile must be a mapping of module name to a mapping of keys, and a profile name must be non-empty.
Rationale: the strict loader fills missing template keys, which would resurrect a deleted starting profile.

### A profile's keys are checked as run_settings are

Decision: the module's `CheckLoad` loads the profiles and runs each through `configreg.PrepareRunSettings`, naming the profile on a refusal.
Rationale: one migrate-then-validate step keeps a profile and a task's run settings to the same rules, and `CheckLoad` is what every `--set` and editor write of the hub file already runs, so a bad profile is refused at the write.

### `--set` takes a JSON object for a whole-entry JSON module

Decision: a new `configengine.Spec` flag, `JSONEntries`, makes `configengine.Set` treat each pair as a top-level entry whose value is a JSON object, replacing the entry whole or appending it, with no template key check;
any other value is refused naming the key.
Only `profiles` sets the flag.
Rationale: the template-shaped set path refuses a new profile name as an unknown key and would re-add every template profile on each write.
Bound: the flag skips the template key check only for the top-level entry name; the written file still passes the module's strict validation and its `CheckLoad`, so a profile with an unknown module or key is refused and the file is put back.

### Removing a profile stays an editor edit

Decision: no remove verb; a rename is a `--set` under the new name plus removing the old in the editor.
Rationale: the board entry names this route, and open-map entries are removed the same way today.

### The task's profile is its own field, not a scope field

Decision: `boardengine.Task` gains `Profile string` (`profile`, omitted when empty), an upsert payload key; an empty string clears it.
`scopeChanged` does not compare it, as it does not compare `RunSettings`.
Rationale: a profile is a run setting by another name, and the run lock already lets a run-held entry take a run settings change.

### The board write checks the profile beside run_settings

Decision: `boardcli`'s run-settings preparation also checks a payload's `profile`: it must be a string, and a non-empty one must name a profile in the hub's `profiles.yaml`, else the write is refused naming the known profiles and the way forward.
A profile change is refused from a task session and carries the running-pair notice, exactly as a run_settings change does.
Rationale: the board entry asks for the refusal at the board write; the task-session guard keeps a task from choosing its own lighter review, as it already cannot change its own run_settings.
Bound: the slug guard is the existing one, guarding a mistaken write and not a determined one.

### Seed-Child layers the task's run_settings over the profile

Decision: battenshed's `ReadRunSettings` seam returns the task's profile and run settings together, and `PrepareRunSettings` takes both.
battencli loads the hub's profiles, refuses an unknown profile as an invalid run setting, and calls a new `configreg.LayerRunSettings`, which prepares the profile and the task settings and merges each module's task document over its profile document through `yamlengine.Merge` with the module's open maps, then validates the merged document again.
The Stuck reason's way forward names the profile field and `lyx config profiles` beside run_settings.
Rationale: the merge is the same one the config layer stack runs, so "layered on top" means what it means everywhere else; reading at Seed-Child alone means a later profile change never reaches a seeded run.

### `lyx board profiles` prints the hub's profiles

Decision: a new board verb, operator audience, prints `{"ok":true,"profiles":[{"name":..., "settings":{...}}]}` in name order, an empty list when there are none.
Rationale: it mirrors `lyx board labels`; a list keeps the order stable in the output.

## Constraints

- `PATTERN-hub-wide-config`: the profiles file is read and written only at `<BoardDir>/_lyx/config/`.
- `PATTERN-config-layers`: only `configengine` reads and stacks the layer files; the loader goes through `StackLayers`.
- `PATTERN-leaf-packages` is not extended: `internal/runprofile` imports only the standard library, `configengine` and `yaml.v3`, but is not declared a leaf.
- `PATTERN-tolerant-records`: the new task field is a known key; no unknown-key refusal is added.
- `PATTERN-refusal-way-forward`: the unknown-profile refusals at the board write and at Seed-Child name the way forward, and a test reaches each.
- `PATTERN-test-economy`: new cases extend the existing table tests of run settings, board upsert and Seed-Child where they cover the behavior.

## Auto-mode assumptions

- The starting profiles in the template are the three the board entry lists, verbatim.
- `lyx config profiles` with no `--layer` addresses the hub file as for every hub-wide module, so `--layer default` is accepted and not required.
- The profile loads use the board dir alone, since the module admits no local key.
- No sandbox suite change: the change adds a verb to the registered `board` module and a config module, not a new lyx module.

## Open risks

- The hub reconcile seeds `profiles.yaml` on the first run after the binary change; a hub that has not run it yet reads the template's profiles.
- A profile key a later binary retires or closes to the local layer stands Seed-Child Stuck, as such a run_settings key does today.

<!-- lyx:carry-over Darn-Review -->
- Darn-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lyx/reviews/darn/round-1-review.md; fixer report: _lyx/reviews/darn/round-1-fixer-report.md.
  - profile-check-untested-on-upsert-batch-and-merge (class design, severity MEDIUM, open in rounds 1)
  - runprofile-load-third-stance-not-in-pattern (class consistency, severity BLOCKING, open in rounds 1)
<!-- /lyx:carry-over Darn-Review -->

## Acceptance criteria

- `lyx config profiles --set <name>='<json>'` adds a new profile and replaces an existing one whole, leaving the other profiles and comments in place; a non-object value is refused; a profile naming an unknown module or key is refused and the file left as it was.
- A profile removed from the hub file stays removed after a load and after `lyx config reconcile --layer default`.
- An absent `profiles.yaml` is seeded with the starting profiles by the hub reconcile.
- `lyx board upsert` with `profile` naming a known profile stores it; an unknown name is refused naming the known profiles; `""` clears it; a profile change from a task session is refused.
- Seed-Child writes the profile's settings with the task's run_settings layered on top per key; a task key overrides the profile's, a profile key the task lacks stays; an unknown profile stands the row Stuck with the way forward.
- `lyx board profiles` prints every profile with its settings, and `[]` when there are none.
- The README entry cell shows the profile.

## Notes for the plan writer

- `internal/modelspec` is the pattern for a seed-only whole-entry module with an embedded `template.yaml`.
