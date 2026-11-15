# A pause holds a run's own processes, not an operator's verbs

A `lym tender pause` waited on every lym process that reached git or GitHub, so operator verbs hung while a pause stood.
Seen: `lym selfreport create`, `lym cleanup`, `lym board get`, `lym machine name`, and the `lym beam start` step of `lym s`.
A pause is often set because something is wrong, which is when those verbs are needed most.
A pause is meant to stop runs, so the network gate now holds only a run's own process, and any other process passes.

Closes #129
Closes #124

## What changes

`tendercli.InstallNetworkGate` takes a new `runCommand` argument and sets `Gate.ExemptAll` for every process that is not a run's own.
Operator verbs are no longer listed one by one as exempt.
A process is held when it is one of these:

- a detached child, as `DetachedChild` already marks it;
- a process whose environment carries `pauserecord.RunProcessEnv` (`LYM_RUN_PROCESS`);
- an agent of a strand that is not an orch, including a strand name that does not parse.

An orch is the operator's delegate and passes.
A passing process still marks its network calls in flight, so a pause's grace still waits for them.

A command is a run command when it carries the new `clihelp.RunProcessAnnotation`.
`cmd/lym/main.go` reads it in `runCommand` and installs the gate through `installPauseGate`.
The annotation is set on `shed run`, `shed step` (so on `batten`, `loom` and the other sheds built from `shedverbs`) and the hidden `beam dobby`.
A run command sets `LYM_RUN_PROCESS` in its own environment, so every lym process it starts is held too.
`reedengine` strips the variable, as it already strips the strand name and parent, before it boots the tmux server.
Otherwise every pane, an orch's included, would be marked as a run's.
`tmuxkit.ServerEnviron` carries the same scrub, and a test pins the two copies together.

The docs of `pauserecord`, `tendercli`, `shedverbs`, `adoptcli` and `hibernateengine` now say a pause holds a run's own processes only.

## Design decisions

- The default is inverted at the one place the gate is installed, so a new operator verb passes without opting out.
- `SetGateExemptAll` stays for an explicit operator verb that runs inside a process a pause would otherwise hold, such as one an agent starts.
- An environment mark, not a process-tree walk, carries "a run started this" to child lym processes.
- The pause's other hold, in `internal/battenshed/innerrun_pause.go`, is unchanged: it holds the spawn of a covered run.
  `lym s` reached it only through the held network call of its `lym beam start` step, which now passes.

## Tests

- `TestRunCommand_MarksOnlyARunsOwnVerbs` pins which commands are run commands, among them `board get`, `board list`, `machine name`, `selfreport create` and the three steps of `lym s`.
- `TestInstallPauseGate_HoldsOnlyARunsOwnVerbs` installs the root's real gate under a live hub pause and checks that operator verbs pass and run commands are held.
- The gate table in `internal/tendercli/cli_test.go` covers operator verbs, an orch, a non-orch agent, an unparsable strand, a run command and a child of a run command.

## To check by hand

- Not done: the idea in #129 to print the held line at once for a verb that does wait.
  A held call still writes its line once, to stderr, when the wait starts.
- Run `lym tender pause`, then `lym selfreport create`, `lym board get <handle>`, `lym machine name <name>` and `lym s` from a covered worktree.
  None should print a held line; `lym tender resume` afterwards.
- Check that an adopt check passes a pause, as the `adoptcli` doc now says.
  A review flagged the earlier doc, which said a pause holds it, and the doc was rewritten to match the gate.
