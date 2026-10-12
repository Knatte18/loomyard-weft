---
round: 2
focus:
  - "Settle whether any card makes a parent lock wait count as activity for the loop's step idle watchdog (`internal/shedverbs/loopwatchdog.go`, `step_idle_timeout_min` in `internal/loomengine/template.yaml`), given that `agentActivityFor` in `internal/loomcli/looparm.go` reads only the verify marker and shuttle waits, so a Merge-In waiting longer than the timeout is not killed as idle."
  - "Settle whether the card wrapping `fabricengine` `Fabric.Pull` for the parent fast-forward enforces, inside the function that moves the pair, that the move is a fast-forward, given that `Pull` in `internal/fabricengine/pull.go` fetches again and hard-resets or re-anchors on a non-fast-forward upstream."
  - "Settle whether the parent lock's unbounded queue wait is covered by the `PATTERN-daemon-wakeups` guard, meaning its file is in the declared list scanned by `cmd/lyx/daemonwakeups_test.go` and its wait is in `pattern/PATTERN-daemon-wakeups.md`."
  - "Settle whether every refusal mapping a card prescribes, including the fabric merge-in-progress mapping of the parent fast-forward, has a named test that reaches it in the package that owns the branch."
---

Entries restate the sites they depend on and do not cap severity.
