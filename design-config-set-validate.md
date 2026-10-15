# lyx config --set stores an invalid model value instead of refusing it

`config-set-validate` · Task · bug, config · running · Darn-Burler-Fix

Imported from [issue #620](https://github.com/Knatte18/loomyard/issues/620).

## Seen

After `config-set-round-list` landed (`4780af4c`), `lyx config loom --layer local --set 'plan=["opus[medium]", "sonnet[high]"]'` reports `ok: true` and writes `plan: '["opus[medium]", "sonnet[high]"]'`, a quoted string, to the layer.
`plan` is a single-model key, not a per-round key, so the value is invalid.
Nothing catches it at write time: the next reader fails, e.g. `lyx loom validate-plan` logs `loom config key "plan": modelspec: whitespace character ' ' at position 16`.
A run started on that layer would fail at load.

Per-round keys work: `plan_review` and `fix` with quoted list entries are stored as real YAML lists, and `webster_fix=[opus, sonnet]` too.

## Expected

`lyx config --set` validates the resulting layer the way the module's loader does, and refuses a value its loader would reject, naming the key.
At least: a list given to a key that takes one model spec is refused, not stored as a quoted string.

## Workaround

None needed; the bad value was written only to a test local layer and removed.
