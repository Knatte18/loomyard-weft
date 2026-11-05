lym webster begin-batch refused

Arguments: 20

```
webster: plan re-resolution at begin-batch reported a blocking finding (plan_conflict: tree_moved): delete-before-reference/22-driftcli-retire-verbs[blocking]: card 22 deletes "internal/driftcli#compareLockRetry", but card 26-drift-retire-skip-keys still references it at internal/driftcli/wiring.go:247 in its Edit code — move the delete to a card after card 26; way forward: edit the plan so the named cards match the tree, run "lym webster rebaseline --card NN" naming each card you edited, then begin-batch 20 again
```

plan_conflict: tree_moved
