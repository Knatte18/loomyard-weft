---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/yamlengine/set.go:226
    summary: the parent's scalar-or-list set rule refuses the task's list of name and model maps, so lyx config loom --set cannot write discussion_advisors
    fix: "In internal/yamlengine/set.go, make scalarOrListNode accept a bracketed value whose entries are scalars or mappings of scalars (drop the entry.Kind != yaml.ScalarNode refusal, keeping the text-after-bracket check), and reword the refusal to say a list of scalars or mappings. Add a row to internal/yamlengine/set_test.go that sets an open-map path whose template value is a list to a flow list holding one mapping with keys name and model, and expects a flow list of mappings, plus a loomengine config test that sets discussion_advisors this way and loads it through LoadConfig."
---

### [MEDIUM] The parent's scalar-or-list set rule refuses the task's discussion_advisors entries

**Location:** `internal/yamlengine/set.go:226` (`scalarOrListOpenMapPaths`, `scalarOrListNode`), `internal/loomengine/config.go:556` (`ConfigOpenMaps`)

**Issue:** The parent's commit `4780af4cb` (`lyx config --set writes a per-round model list as a list`) makes every declared open-map path whose template value is not a mapping a scalar-or-list path.
For such a path, `SetValues` and `SetSparse` route the value through `scalarOrListNode`, which refuses any list entry that is not a scalar.
`ConfigOpenMaps` still lists `discussion_advisors`, and its template value is `[]`, so it is now one of those paths.
The task (D7) changed the key's entries from strings to `{name, model}` mappings.
Before the merge, `replaceSequence` accepted any YAML list, so `lyx config loom --set 'discussion_advisors=[{name: critic, model: "sonnet[high]"}]'` wrote the key.
After the merge the same command is refused with "neither one value nor a YAML flow list of scalars ... quote each entry that holds [ or ]", which misdirects the operator.
A non-bracketed value is written as a string scalar, which `DiscussionAdvisorList.UnmarshalYAML` then refuses at load.
So the only way to name advisors is to edit `loom.yaml` by hand, although D7 says the operator chooses advisors in config.
Neither side's tests cover this: the parent's `set_test.go` rows use scalar lists, and the task's config tests load files and never go through `--set`.
The task's own refusal way forward ("set the key to a list of {name, model} entries") names no command, so it does not contradict the merged code, but the natural command no longer works.
