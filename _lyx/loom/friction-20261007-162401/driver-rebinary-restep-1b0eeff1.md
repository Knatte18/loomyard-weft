# Driver automatic re-step: binary change

```yaml
kind: binary-change re-step
stop: Webster blocked, "webster master is asking a question" (session c49f023a-9153-4106-ac1f-0444b0d813d8)
trace_file: /home/knatte/Code/loomyard-LYXHUB/shuttle-turns/.lyx/logs/trace-20261007T140739Z-1b0eeff1c7439870-2544822.log
justifying_field: last_step.binary_changed=true
build_identity_before_restep: vcs_revision 98c3bf057fb130f377a4fb52b9f25008999c7969
clean_tree_guard: git status --porcelain empty
action: remove park marker, re-step once
outcome: see the stop report
```

The master's last message said batch 13 failed an audit because the fork ran a command that reads outside the worktree, and that `recover-batch 13` was running in the background.
