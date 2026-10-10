---
round: 3
focus:
  - "Check how the `help` command, which the decision record makes a group with an `index` child, meets the `TestCLITree_EveryCommand` walk in `cmd/lyx/clitree_test.go`: bare `lyx help` must exit 0 with output naming every visible child, and `lyx help bogus` must exit 1 with an error containing `unknown subcommand`; settle whether the record states either a conforming behaviour or a named exemption with its reason, and whether the audience and form tests still reach `help` and `index`."
  - "Check which binary the orch watcher's index render describes: `spawnWatcherProcess` in `internal/orchcli/spawn.go` starts `orch watch` with the environment of the `lyx orch start` process, while the orch pane's `lyx` comes from the reed pane prelude in `internal/reedengine/panebin.go` (`LYX_BIN`); settle whether the record names one binary for the watcher, start, adopt and the `resume-context` hook, and states what bounds the render when that binary is missing."
  - "Check whether the record states how `lyx help <group> <unknown-word>` resolves, given cobra's `Find` returns no error for a known group followed by an unknown word, and whether an acceptance criterion covers it beside the unknown-topic case."
---

Each entry restates its sites so the reviewer needs no other context.
