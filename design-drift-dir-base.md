# drift.yaml joins dir onto the anchor, gate.yaml onto the worktree root

`drift-dir-base` · Note · bug, drift, gate, config

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
