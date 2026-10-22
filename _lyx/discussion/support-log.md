# Support log — config layers set one profile key and merge profile maps by name

## Interview

A darn run holds no interview: the writer read the board entry, which imports issues #833 and #834, and the code of `configengine`, `yamlengine` and `batcher`.

## Rejected alternatives

- Change open maps to merge by name in `yamlengine.Merge` and `FillMissing`: every open map in every module would change its layer behaviour, including scalar-or-list open maps such as loom's model keys.
- Drop the `profiles` declaration altogether, leaving a plain template mapping: the load would merge by name, but prune and the unknown-key warning would drop or warn a profile the template lacks, and `--set` could not add one.
- A new `yamlengine` map kind beside open and per-name maps: the per-name map's load, prune and reconcile behaviour is already the wanted one, so only its set and the local check need the mapping-entry case.
- Keep the local and machine checks unchanged: a run profile or local file adding a batcher profile, which the open-map declaration admitted, would start being refused.

## Review rounds

_No rounds yet._

## Question ledger

- Should a hub layer be able to hide a template profile? Self-pick: no; the board entry asks that layers add to or override by name, and no layer deletes a key from the one below.
- Should `--set` reach leaves of a profile the template lacks? Self-pick: yes, for every path some template profile holds, so a hub-added profile is tuned the same way as a template one.
- Which file does the retired-key refusal name for a profile merged from several layers? Self-pick: the highest layer holding the profile name.
