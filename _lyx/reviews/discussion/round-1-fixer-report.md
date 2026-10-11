# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed (the denylist route).
  Premise confirmed: `reed.shell` is run by `exec.Command(e.cfg.Shell, ...)` (`internal/reedengine/proctree_windows.go`), `reed.tmux_conf` is sourced into the hub server (`internal/reedengine/lifecycle.go`), `shuttle.claude` is the agent executable, the `claude_deny_*` keys gate `standingDenies` (`internal/shuttleengine/claudeengine/settings.go`), and `shuttle.run_dir` must stay worktree-local (`internal/shuttleengine/template.yaml`).
  D4 now adds `configreg.Module.LocalDenied` for per-worktree modules (reed: `shell`, `tmux`, `tmux_conf`; shuttle: `claude`, `run_dir`, every `claude_deny_*`), states the rule for a future key, rewrites the bound, and limits the board-history claim to what Seed-Child writes.
  Followed through in D5, D8, the `PATTERN-config-layers` constraint, the auto-mode assumption, R3, the load and board-upsert acceptance criteria, and the support log's Q5 and Rejected alternatives.
- F2 (BLOCKING), fixed as proposed.
  Premise confirmed: `configreg.Module.SeedOnly`'s doc and the `m.SeedOnly` branch of `internal/configsync/configsync.go` never rewrite a present file, and `burlerengine/template.yaml` and `modelspec/template.yaml` declare both files operator-owned.
  D2 now leaves every seed-only layer untouched by reconcile, apart from D6's one-time strip, and states how its entries merge under D1.
  D5 exempts a seed-only local file from the template-lacks refusal, D8 accepts a seed-only key path and checks shape only where the template holds the path, and the reconcile, load and board-upsert acceptance criteria carry the exception.
- F3 (MEDIUM), fixed as proposed.
  D6 now states that the strip turns a value equal to the seeded default into an inherited one that a later default change reaches, and bounds it with a `stripped` list per worktree and module on the reconcile result.
  D7's reconcile output and the migration-walk acceptance criterion carry the `stripped` report.
- F4 (LOW), fixed as proposed.
  D9 now names what the removal can discard, a local file written into the pair while Seed-Child was Stuck, and that the board entry's `run_settings` is the only override surviving the resume.
  Premise confirmed against `internal/battenshed/create.go`'s pre-existing-branch refusal.
- Support log: round 1 recorded under `## Review rounds`.

## Disputed

None.

## Deferred

None.
