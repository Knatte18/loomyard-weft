# Support log — run-profiles

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- Profiles as keys inside `run_settings`: the board entry's operator decisions put the profile in its own field.
- The template-shaped `--set` path for profiles: it refuses a new profile name as an unknown key and re-adds every template profile on each write.
- Loading profiles through the strict `configengine.Load`: its template fill would bring back a deleted starting profile.
- A plain recursive map merge of the profile and the task settings: it would differ from the config layer stack's merge on open maps, so `yamlengine.Merge` with each module's open maps is used instead.
- Folding the profile into battencli's `ReadRunSettings` closure alone: an unknown profile would stand the row Stuck with no way forward, so the seam carries the profile to the prepare step, which routes it as an invalid run setting.
- Comparing the profile in the run lock's scope check: a profile is a run setting and the run lock lets run settings changes through.

## Review rounds

_No rounds yet._

## Question ledger

- Q: Does `--set` need `--layer default`? Self-pick: no; a hub-wide module's file is the default target, and `--layer default` addresses the same file. Resolved.
- Q: May a task session change its own profile? Self-pick: no, the run_settings guard covers it, since a profile lowers review rounds as run_settings do. Resolved.
- Q: Where does the README show the profile? Self-pick: as a code-span bullet in the entry cell before the run settings lines. Resolved.
- Q: What does a hub with no `profiles.yaml` read? Self-pick: the template's profiles, as other modules degrade to their template. Resolved.
