# Binary-change re-step: Webster

```yaml
run_id: lifecycle-waits
stop: blocked at Webster, history_length 23
justification: last_step.binary_changed true (vcs_revision dc30e1a98d40f861e49298491942741976b87940 before the re-step)
clean_tree_guard: passed
action: one re-step of `lyx shed step lifecycle-waits`
outcome: Webster done, history_length 24, next Webster-Bouncer
```

The stop was `record-batch` refusing batch `01-orch-compact-boundary` on a stale `startup_context` key in `batcher.yaml`; `ly:orch` fixed the pair's `_lyx/config/batcher.yaml` meanwhile.
