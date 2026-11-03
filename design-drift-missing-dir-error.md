# A Drift step in a missing dir fails as fork/exec /usr/bin/sh, with a way forward that repeats it

`drift-missing-dir-error` · Note · bug, drift

A Drift capture whose `dir` does not exist fails as `fork/exec /usr/bin/sh: no such file or directory`, and its way forward repeats the same failure.

## Seen

On the `cs` hub (2026-11-03), the baseline capture of `pipeflow-reynolds-helper` ran with a `dir` that resolved to a missing directory (see the note on Drift's `dir` base).
The capture record and the Drift-Await stop read:

```
drift: the baseline capture ended failed (drift: start the drift capture command in /home/knatte/Code/lym-csharp-LYMHUB/pipeflow-reynolds-helper/dotnet/dotnet: fork/exec /usr/bin/sh: no such file or directory), log …/capture-20261103T142802.258Z.log; way forward: lym drift capture followed by lym loom resume, … or lym drift defer followed by lym loom resume
```

- The text blames `/usr/bin/sh`, which exists; Go reports a failed `chdir` in the child as a failed exec of the shell.
- `lym drift capture` again runs the same config and fails the same way; the way forward names neither the config nor where it lives.
  The capture log is empty.
- The batten notice to the orch shortened the directory to `in dotnet`, hiding the doubled path.
- The run's agent asked the orch where the capture directory is configured, "since I could not find it in the repo": `drift.yaml` sits in the hub's `_bolt/_lym/config`, outside the task worktree.

`internal/gatecli` refuses the same case by name: "could not start the shell for … in <dir> …; way forward: make the tier's dir an existing directory".

## Expected

- Before it spawns, Drift checks that the step's resolved `dir` exists, and a missing one fails with the resolved path, the `drift.yaml` key that set it, and the way forward `lym config drift --set <step>.dir=<dir>` from the prime, then `lym drift capture` and `lym loom resume`.
- The batten notice keeps the full resolved path.
