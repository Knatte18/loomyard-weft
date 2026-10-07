# Driver record: binary-change re-step

```yaml
run_id: card-batching
stop: blocked at Webster, history_length 21, trace a8ca171d439f1bef
justifying_field: last_step.binary_changed true
build_before: vcs_revision d536a1a4edaa5f55d999282207b8bdfd75ca698e
build_after: vcs_revision 9ba812fcc814324d3f3fab11f360d49545d3e32e
action: removed the park marker, ran one `lyx shed step card-batching` (trace ddaac7f58c8d3330)
outcome: blocked again at Webster, history_length 22, same cause
```

The re-step trace logged `websterengine: stopping a leftover live strand before respawning it` for two strands, then the master asked again.
Its last message: `record-batch 8` came back `batch_failed` because the fork ran `tokencount` with `-history ../card-batching-weft`, which the audit treats as a correctness violation; the master started `lyx webster recover-batch 8` and waited for it.
