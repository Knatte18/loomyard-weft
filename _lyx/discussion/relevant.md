# Relevant files

- `internal/fabricengine/doc.go` — documents the origin reconcile's scopes, entry states, moves and seams that D4 extends
- `internal/battencli/resumereconcile.go` `reconcileRun`, `reconcileResume`, `resumeRefusal` — batten's resume refusal that D3 extends with the owner refusal
- `internal/hibernatecli/reconcile.go` `wakeReconcile`, `wakeReconcileFromReport` — the wake's mapping of entries onto pauses that D4 extends
- `internal/hibernateengine/doc.go` — the wake's pause of stopped runs and its owned pause entry, which D4's marking follows
- `internal/orchcli/start.go` `reconcileHub`, `reconcileNote` — orch start's reconcile that D4 makes pause owned-elsewhere runs
- `internal/orchcli/doc.go` — orch start never refuses on the reconcile
- `internal/cleanupcli/doc.go` — the composite top-level verb shape `lyx adopt` copies
- `internal/boardengine/task.go` `Task` — the board record the owner field joins
- `internal/boardengine/runlock.go` `RunLockedError`, `IsRunStatus` — the run lock the owner field must pass
- `internal/battenshed/doc.go` — InnerRun's sleep and pause reading that D6 builds on, and Open-Window's resume rule that D8 changes
- `internal/battenshed/land.go` — the Land row that follows the parent-side merge
- `internal/battenshed/template.yaml` — Open-Window's switch and command
- `internal/battencli/cli.go` — `--window` and the hidden `--open-terminal` flag
- `internal/landingshed/doc.go` — Finalize and the shared parent guard where D7's check sits
- `internal/configcli/doc.go` — the machine layer, rejected as the name source
- `cmd/lyx/main.go` — the root command list `lyx adopt` joins
