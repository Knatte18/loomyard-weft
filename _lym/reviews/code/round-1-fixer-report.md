---
dispositions:
  - id: F1
    disposition: ruling
    reason: The parent refused the transition window; reed's revival now drops the remote-control argument while orch.yaml has remote_control off.
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

## Changes

- F2: `_lym/landing/summary.md` now says the walk's `ReconcileHubWideAt` pass seeds an absent orch.yaml first, so the reinterpretation reads it and flags it when it sets no `remote_control`.
  The file lies under `_lym`, so it is edited but not committed.
- F3: `internal/configreg/configreg_test.go` gains `TestModules_OrchRegistersRemoteControlReinterpretation`.
  It asserts the orch module holds exactly one reinterpretation with the literal ID `orch-remote-control-off-by-default` and a non-nil `Rewrite`.
  Commit 7cfd66753; `lym gate test ./internal/configreg` passed.
- Gate repair: the new test's two-line doc comment wrapped one sentence across lines;
  it is now two sentences, one per line, each under 120 characters (commit 8634af210).
- Self-check follow-up: the summary's "Tests" section gained a bullet for the new test.
- F1 ruling (commit 2f3cbdd7b): see Rulings.
- Gate repair: the doc comment of `remoteControlArgument` in `internal/orchengine/revival.go` reached 120 characters;
  it is now two sentences on two lines.
- Gate re-check: a tier2 verify reported `internal/reedcli` failing to build with `undefined: orchengine` at `internal/reedcli/cli.go:115`.
  The committed `cli.go` imports `orchengine`, `go vet ./internal/reedcli` passes, and `lym gate test --tier 2 ./internal/reedcli` passes.
  The verify most likely read the file between the edit that added the call and the one that added the import, so no change was needed.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the summary's "Tests" section omitted the new test; fixed as proposed.
- fix-proposal: no findings.
- fix-affected-set: LOW, `internal/hubreconcile/reinterpret.go:45` says an absent base file writes only its marker;
  fixed differently by leaving it: the sentence describes `applyReinterpretations` in isolation, which still holds, since seeding happens in the walk before it runs.

The F1 ruling's change arrived after the self-check and was not put through the fork lenses.

## Disputed

None.

## Deferred

None.

## Rulings

- F1: the parent refused the transition window, so no launch, resume or revival turns remote control on unless the hub's config asks.
  Carried out in full, and no flag to the parent was needed.
  - `reedengine.Engine` gains `SetRevivalFilter`, and `revivalCmd` picks the stored resume line, else the command, and runs it through the filter.
    Reed's `Resume` (`internal/reedengine/lifecycle.go`) and its absent-pane relaunch (`internal/reedengine/strand.go`) both replay that line.
    Reed still reads no config file: the caller injects the filter.
  - `orchengine.RevivalFilter` (`internal/orchengine/revival.go`) reads orch.yaml at each call and strips the ` --remote-control <name>` pair unless `remote_control` is on.
    An orch.yaml that does not load cannot ask for remote control, so the pair is dropped then, with a Warn log.
  - The filter is wired where reed's `Resume` and `reed add --if-absent` run: `internal/reedcli/cli.go` and the task-worktree revival in `internal/battencli/wire.go`.
  - `internal/orchengine/revival_test.go` pins the strip for POSIX and pwsh quoting and bare names, and the filter for no file, explicit off, explicit on, and an on without a name.
  - `lym gate test` over `orchengine`, `reedengine`, `reedcli` and `battencli` passed.
  - Not tested: reed's replay through the filter, since `Resume` needs a live tmux.
