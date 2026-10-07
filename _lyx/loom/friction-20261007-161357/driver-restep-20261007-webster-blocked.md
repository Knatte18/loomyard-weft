# Binary-change re-step record

```yaml
stop: Webster, state blocked, "webster master is asking a question" (trace 887b45bb26de140c, session 39aea993)
field: last_step.binary_changed true; build before the re-step vcs_revision 98c3bf057fb130f377a4fb52b9f25008999c7969, after it ca87714220ce338b4e99d6871b5beeb79e33fdbf
clean_tree_guard: git status --porcelain in the code worktree listed nothing
action: removed the park marker (none present), ran one `lyx shed step test-tiers`
outcome: Webster blocked again (trace 09923d687af88359, session a8aa0a27); the master holds `record-batch 10` until the batch 10 fork's final notification, the fork still waits on background measurements
```

Trace lines acted on:

- `websterengine: master run is asking ... Batch 10's fork reports it is still running its after-state measurement`
- `websterengine: stopping a leftover live strand before respawning it; an in-flight batch run restarts from state.json rather than resuming`
