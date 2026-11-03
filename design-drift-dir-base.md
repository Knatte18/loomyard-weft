# One base for a command dir; a missing Drift dir fails by name

`drift-dir-base` · Task · bug, drift, gate, config · running · Darn-Write

Depends on: [`gate-orphan-pipe-hang`](design-gate-orphan-pipe-hang.md)

# One base for a command's dir

A command's `dir` means a different base in `drift.yaml` than in `gate.yaml`, so the same value works in one file and breaks the other on a hub anchored at a subdirectory.

## Seen

On the `cs` hub (anchor `lym-csharp/dotnet`, 2026-11-03), `gate.yaml` and `drift.yaml` both set `dir: dotnet`.

- The gate ran its build and tiers in `lym-csharp/dotnet`: `internal/gatecli/tierrun.go` `resolveRunDir` joins `dir` onto the worktree root, as `internal/gatecli/doc.go` states ("The tier's `dir`, relative to the worktree root").
- Drift ran the baseline capture of `pipeflow-reynolds-helper` in `pipeflow-reynolds-helper/dotnet/dotnet`, which does not exist: `internal/driftengine/steprun.go` joins `dir` onto `run.AnchorDir`, as `internal/driftengine/doc.go` states ("A step's relative `dir` is joined onto the worktree's anchored directory").
- The run blocked at Drift-Await; its agent asked whether to set the dir to `dotnet/`, which is the value that broke it.

`lym config drift --set capture.dir=dotnet` accepted the value with no sign of which base it resolves against.

## Expected

- One base for `dir` across the hub's command configs: `gate.yaml`'s tiers and `build`, and `drift.yaml`'s `capture` and `compare` resolve a relative `dir` against the same directory.
- Each module's config template says which base that is, next to the `dir` key.
- `lym config <module> --set` and the config loader refuse a `dir` that does not exist in the prime, naming the resolved path.
- Existing hub configs keep working: the `cs` hub's `drift.yaml` now sets `dir: .` against the anchor; whichever base is chosen, the change says how such a value carries over.

# A missing Drift dir fails by name

A Drift capture whose `dir` does not exist fails as `fork/exec /usr/bin/sh: no such file or directory`, and its way forward repeats the same failure.

## Seen

On the `cs` hub (2026-11-03), the baseline capture of `pipeflow-reynolds-helper` ran with a `dir` that resolved to a missing directory.
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
