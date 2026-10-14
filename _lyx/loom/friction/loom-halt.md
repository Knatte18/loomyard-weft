loom halted blocked at Webster

```
begin-batch 17 refused with plan_drifted: delete-before-reference/18-retired-builders-delete[blocking]: card 18 deletes "internal/loomengine#PlanSpec", but card 19-loom-config-axes still references it at internal/loomengine/plan_test.go:79 in its Edit code — move the delete to a card after card 19; way forward: edit the plan so the named cards match the tree, run "lyx webster rebaseline --card NN" naming each card you edited, then begin-batch 17 again
```

producer: Webster
state: blocked
anomaly: escalation-to-human
history_entries: 20
trace_file: /home/knatte/Code/loomyard-LYXHUB/overview-seats/.lyx/logs/trace-20261014T180442Z-efb82da6ec3fb3db-3169486.log

history:
- Preflight / done / 2026-10-14T16:48:54Z
- Loom-Preflight / done / 2026-10-14T16:49:06Z
- Discussion-Write / done / 2026-10-14T17:00:57Z
- Discussion-Bouncer / stuck / 2026-10-14T17:01:34Z
- Discussion-Burler-Review / stuck / 2026-10-14T17:07:28Z
- Discussion-Burler-Fix / stuck / 2026-10-14T17:08:48Z
- Discussion-Bouncer / stuck / 2026-10-14T17:09:54Z
- Discussion-Burler-Review / stuck / 2026-10-14T17:13:31Z
- Discussion-Burler-Fix / stuck / 2026-10-14T17:15:01Z
- Discussion-Bouncer / done / 2026-10-14T17:15:46Z
- Plan-Write / done / 2026-10-14T17:44:09Z
- Plan-Bouncer / stuck / 2026-10-14T17:44:43Z
- Plan-Burler-Review / stuck / 2026-10-14T17:54:51Z
- Plan-Burler-Fix / stuck / 2026-10-14T17:57:08Z
- Plan-Bouncer / stuck / 2026-10-14T17:57:46Z
- Plan-Burler-Review / stuck / 2026-10-14T18:02:21Z
- Plan-Burler-Fix / stuck / 2026-10-14T18:03:36Z
- Plan-Bouncer / done / 2026-10-14T18:04:18Z
- Batchifier / done / 2026-10-14T18:04:30Z
- Webster / stuck / 2026-10-14T18:51:56Z
