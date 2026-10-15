# Round 2 fixer-report — discussion

## Changes

- **F1** (MEDIUM, design), fixed as proposed.
  Premise checked: `TestModules_EveryTemplateKeyClassified` refuses a `Categories` entry naming no template path (`internal/configreg/configreg_test.go:96-99`), `claude_prompt_cache_ttl_roles` is in `shuttle`'s `OpenMaps` (`internal/shuttleengine/config.go:57-58`), and `ValidateLocal` refuses a unit absent from the template (`internal/configengine/layers.go:168-169`).
  In D4's retired-key paragraph of `_lyx/discussion/decision-record.md`, the field's shape is now stated: a map from each retired dotted key to the category it held, with an open-map flag; `ValidateLocal` admits a unit a `CategoryNone` retired key equals or prefixes and refuses one covered by a retired key of any other category, naming the key; the retired keys leave `Categories` and `OpenMaps`.
  The first sentence's "admits a retired key a local file holds" now carries "when its category allows it", so it no longer reads as unconditional.
  The acceptance row beginning "Before any reconcile" gains: a local `claude_prompt_cache_ttl_roles` holding two roles loads, and a local `claude` or `claude_deny_agent_tool` is still refused naming the key.

## Disputed

None.

## Deferred

None.
