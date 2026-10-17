---
round: 2
focus:
  - "PATTERN.md and pattern/ diff against the merge base: does any entry other than PATTERN-recipe-setting-lookup differ from its merge-base text, and does PATTERN-recipe-setting-lookup carry exactly the declared text?"
  - "internal/shedrecipe/entries_settings.go fanSetting: how does a fan list holding a null element decode, and does a test row pin that behavior?"
  - "internal/shedrecipe/entries_settings.go refusals for a malformed module.key reference, a wrong-shape fan setting and a non-bool auto-accept setting: does each have a test asserting its message?"
  - "internal/shedrecipe/entries_burler.go FixFanKey: is the value a bare config key, as the burlerengine fix refusals that splice it expect?"
  - "internal/loomcli/wiring.go comment on loomSegmentPerches and loomPerches: does it describe only lookups that still exist?"
---

Targets the fixes for the five round-1 findings; each entry restates its site and question.
