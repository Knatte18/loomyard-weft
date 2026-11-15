# A pause holds only a run's own processes; operator verbs pass

A `lym tender pause` held every lym process in the hub that reached origin, because the root pre-run installed one network gate per process and every git and GitHub call waits in it.
So `lym selfreport create`, `lym board get`, `lym machine name`, `lym cleanup` and the `beam start` step of `lym s` waited on a pause that has nothing to do with them, often when they were needed most.
This change inverts the default: the gate holds only the processes a pause is meant to stop, and every other process passes every live entry.

## What a pause now holds

`tendercli.InstallNetworkGate` sets `Gate.ExemptAll` unless the process is a run's own, which is one of:

- a command marked with the new `clihelp.RunProcessAnnotation`: the shared `run` and `step` verbs of `internal/shedverbs` (so `lym batten run|step`, `lym loom run|step`, `lym shed run|step`) and the hidden `lym beam dobby`;
- a lym process such a command started, directly or further down; the marked command sets `pauserecord.RunProcessEnv` (`LYM_RUN_PROCESS`) in its own environment, which its children inherit;
- a detached push or board-sync child, as `DetachedChild` already marks;
- an agent of a strand whose role is not `orch`, read from `LYM_STRAND_NAME`; a strand name that does not parse is held.

Everything else, an operator's verb from a terminal or a script, or the orch acting for the operator, passes.
Its calls are still marked in flight, so a pause's grace still waits for them.
`cmd/lym/main.go` decides through `runCommand(cmd)` where it installs the gate, so no verb opts out one by one; `SetGateExemptAll` stays for an explicit verb run inside a held process, such as `lym adopt` from an agent.

`reedengine` strips `LYM_RUN_PROCESS` from the tmux server's environment beside the strand name and parent (`stripAgentNameEnv` is renamed `stripIdentityEnv`), so a server booted by a held process does not mark the orch's pane.
`tmuxkit.ServerEnviron` strips it too, and the test pinning the two filters together carries the key.

## Design decisions

- The hold is decided by process kind, not by verb list: two annotated command groups plus inheritance cover the dobby, the batten and loom loops, their step children, gate runs and the testmap upkeep they spawn.
- The orch is not held. A pause never interrupts the orch, and it is the operator's delegate, so its verbs pass like the operator's.
- The second hold the board entry names, `battenshed`'s spawn hold of a covered run, and the run-entry waits in `battencli`, `loomcli`, `shuttleengine` and `verifytree` are unchanged: each sits inside a run.
- Making a hold visible needed no change: a held call already writes its line through `Notify` at the wait's first check, before any sleep.

## Tests

- `TestRunCommand_MarksOnlyARunsOwnVerbs` (`cmd/lym`) pins that `board get`, `board list`, `machine name`, `selfreport create` and the three steps of `lym s` (`adopt`, `beam start`, `reed attach`) are not run commands, and that the shed `run`/`step` verbs and `beam dobby` are.
- The table in `TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` gains rows over a live pause: an operator verb passes a hub entry and its own pair's entry; the orch passes; a pair agent, an unparsable strand, a run command and a process carrying the mark are held; a run command sets the mark.
- `TestStripIdentityEnv_DropsNameParentAndRunMarkOnly` covers the reed strip.

## Open risks

- `lym board get` and `lym machine name` hung with no held line seen, so their hang may not have been the gate. If it was some other wait, this change does not touch it.
- A run-owned process that the shed verbs or dobby do not start, and that runs in no strand, now passes a pause. No such process was found, but a new one would need the annotation.
