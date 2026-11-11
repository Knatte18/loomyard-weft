# Round 1 fixer-report

## Changes

- F1 (fixed as proposed, `internal/shuttleengine/codexengine/sandbox.go`):
  `runSandboxProbe` wraps the new `errSandboxProbeNotRun` when the probe could not run to a verdict
  (context done, error not an `*exec.ExitError`, or a negative exit code from a signal).
  `launchSandboxMode` refuses the launch under `auto` and `workspace-write` for that kind, and degrades `auto` only for the executable's own non-zero exit.
  Rows added: `TestRunSandboxProbe` `ExecutableMissing` and `ProbeSignalled`, and `TestPrepare_SandboxMode` `AutoProbeDidNotRun` and `WorkspaceWriteProbeDidNotRun`.
  The refusal's way forward names fixing the executable or launching again, not `danger-full-access`, so the write bound is never dropped on advice for a transient failure.
- F2 (fixed as proposed, `open_test.go`): `TestOpenLaunch` row `WorkspaceWriteSandboxCannotStart`;
  the probe stub records `bin`, `home` and `dir`, asserted for rows with `wantProbe`.
  The unmade-home check is skipped for a `wantProbe` row, since the probe runs under the lym home and so after it is made.
- F3 (fixed as proposed, `sandbox.go`): `runSandboxProbe`'s doc comment states the probe does not run the launch's writable roots or network setting, and that an empty dir runs it in the process's working directory.
- Self-check fixes: `doc.go`, `config.go`, `template.yaml` and `docs/overview.md` now state the did-not-run refusal;
  the over-long doc line is split and no longer names another symbol;
  the refusal message no longer repeats "did not run to a verdict".

Gate findings repair: five comment lines in `config.go`, `doc.go` and `sandbox.go` that wrapped mid-sentence or reached 120 characters
are rewritten as one sentence per line; no code changed.

Commits: F2, F1 with F3 (both change `runSandboxProbe`'s doc and share its test files), and one for the self-check fixes.
Gate: `lym gate test ./internal/shuttleengine/codexengine --tier 0,1` passes.

## Self-check

- fix-code-correctness, LOW, signal branch untested: fixed (`ProbeSignalled` row).
- fix-code-correctness, LOW, dead `wantErr` on `ExecutableMissing`: fixed (replaced by `wantNotRun`).
- fix-code-correctness, outside lens, over-long doc line: fixed.
- fix-consistency, MEDIUM, `doc.go`, `config.go`, `template.yaml`, `docs/overview.md` stale: fixed.
- fix-consistency, LOW, way forward suggests `danger-full-access`: fixed differently from the first draft (advice dropped).
- fix-consistency, LOW, `wantProbe` doubles as a switch: fixed (field comment says so).
- fix-proposal, LOW, dead `wantErr`: duplicate of the above, fixed.
- fix-proposal, LOW, doc names a sentinel symbol: fixed (reworded).
- fix-proposal, LOW, repeated message text: fixed.
- fix-proposal, LOW, home assertion weakened for the probe row: kept; the probe needs the home, so the check cannot hold for that row.
- fix-affected-set: no findings.

Lens reports: `round-1-fixer-report.lens-<lens>.md` beside this file, one per lens; all four returned.

## Disputed

None.

## Deferred

None.

## Rulings

None.
