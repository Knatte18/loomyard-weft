---
round: 2
focus:
  - "internal/yamlengine/set.go: `SetValues` and `SetSparse` route a value for an open-map path whose template value is not a mapping through `scalarOrListNode`. Settle whether a flow list of `{name, model}` mappings, as in `lyx config loom --set 'discussion_advisors=[{name: critic, model: \"sonnet[high]\"}]'`, is now written as a list of mappings, and whether scalar-list and single-scalar values behave as before."
  - "internal/loomengine/config.go `ConfigOpenMaps` lists `discussion_advisors` with template value `[]`, and `DiscussionAdvisorList.UnmarshalYAML` accepts `{name, model}` mappings. Settle whether a value written through `--set` loads through `LoadConfig` into the same advisors, and whether the refusal text for a malformed bracketed value still names a correct way forward."
  - "Settle whether the tests cover the interaction: a `yamlengine` set test with a mapping-entry list on an open-map path, and a `loomengine` config test that sets `discussion_advisors` through the set path and loads it."
---

The focus targets the interaction of the parent's scalar-or-list set rule with the task's mapping-valued `discussion_advisors` entries.
