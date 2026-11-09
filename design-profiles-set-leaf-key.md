# lym config profiles --set cannot reach a profile's leaf key, and its refusal names the wrong value shape

`profiles-set-leaf-key` · Task · bug, config, profiles · running · Worktree-Create

Imported from [issue #94](https://github.com/Knatte18/loomyard/issues/94).

## Seen

In the DrillOpPlan hub's prime, changing two values of the `docs` profile's darn loom block with `lym config profiles --layer default --set` took three tries.

1. `--set docs.darn.loom.code_max_bounces=0 --set docs.darn.loom.code_hard_max_bounces=1` was refused:
   `value for docs.darn.loom.code_max_bounces must be a JSON object such as {"module": {"key": 1}}, got "0"`.
   The message names a value shape that fits a recipe block, not a leaf key.
2. `--set 'docs.darn={"darn":{"writer":"sonnet[medium]"},"drift":{"scope":"off"},"loom":{...}}'`, which is the `{"module": {"key": 1}}` shape the first message asked for, was refused:
   `profile "docs.darn": recipe "darn": module "writer" must be a mapping of keys`.
   The value was read one level deeper than its key.
3. `--set 'docs={"darn":{"darn":{"writer":"sonnet[medium]"},"drift":{"scope":"off"},"loom":{"code_max_bounces":0,"code_hard_max_bounces":1,"code_reviewer_fan":""}}}'` worked.

## Workaround

Set the whole profile, keyed by the profile name alone, with every block it holds restated.

## Expected

`--set` reaches a leaf key of a profile, as it does in other modules (`lym config gate --set tiers.tier0.env.X=...`), so `docs.darn.loom.code_max_bounces=0` sets that one value.
Failing that, the refusal names the key depth `--set` takes and a value shape that is accepted at that key.
