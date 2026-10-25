# Add Drift-Baseline, Drift-Await and Drift-Verify regression rows

A hub can now run a regression suite around a task: capture profiles on the task's branch point before any code changes, compare after the implementation, and let a strong agent judge and fix the drift.
Regression is no test tier.
No gate runs it, it is never narrowed or carried, and its outcome is a drift judgement, not pass or fail.
The hub-wide `drift.enabled` flag is `false` by default, and with it off every Drift row records `Skipped` and spawns or runs nothing, with no recipe overlay.
This hub keeps it off.

Addresses issue #609's regression suite want; the board entry lists no issue to close.

## What changes

- New module `internal/driftengine` owns the three producers, the hub-wide `drift` config module, the capture and compare runs and their records.
  New `internal/driftcli` wires the `lyx drift` subtree: `scope`, `capture`, `compare`, `accept`, `withdraw`, `defer` and `waive`.
- Loom gets `Drift-Baseline` after the Discussion perch, `Drift-Await` before `Webster`, and `Drift-Verify` after the Code perch, before `Describe`.
  Darn gets `Drift-Baseline` after `Preflight` and `Drift-Verify` before `Code-Bouncer`; it has no `Drift-Await`, because its baseline capture runs in the foreground.
- `drift` config keys: `enabled`, `capture` and `compare` (`command`, `dir`, `env`, `timeout_min`), `baseline_model`, `verify_model`, `verify_max_runs`.
  A declared command must contain `{baseline_dir}`; `{skip_file}` is optional.
  With the flag on, an undeclared command is refused, naming `lyx config drift`.
- `Drift-Baseline` runs a Sonnet-class agent that records the scope (`off`, `partial` with `--skip` tests, or `full`) in `_lyx/drift/scope.json` and its expected drift in `_lyx/drift/baseline.md`, then starts the capture.
- `Drift-Await` is mechanical: it waits while the capture holds its lock and halts on a failed, dead or missing capture, or on HEAD having moved.
- `Drift-Verify` compares mechanically.
  A passing compare spawns nothing.
  A failing one spawns one agent on `verify_model` that accepts acceptable drift, traces and fixes the rest, and writes `_lyx/drift/verify-report.md`.
  Its fixes are checked by the new `drift-verify` gate key, then compared again.
- Supporting changes in existing code: a `capture` role in `runproc`, `agentActivityFor` counting the capture lock and compare marker as activity so the step idle watchdog never kills a wait, and `pairteardown` ending the capture on cancel and adopt.
  Two stencils for the agents, the recipe rows and registry entries, and docs (`doc.go`, `docs/overview.md`, the `pattern/` lists of gate-slot holders, verify sites and run-process roles).

No existing producer changes behaviour.

## Design decisions to review

- **The capture runs in the task worktree.**
  It refuses unless HEAD is the branch point and tracked files are clean, so it never sees changed code.
  It holds an OS lock for its whole life; readers probe it with a non-blocking take, and a capture start retries its own take for a few seconds, so a probe never refuses a capture.
- **A baseline is never deleted or overwritten.**
  A non-empty baseline directory is renamed to a numbered backup first.
  A capture passes only on exit code 0 with at least one file written.
- **Acceptance without touching the baseline.**
  `lyx drift accept` appends to the scope record's accepted list, and every later compare skips those tests.
  `lyx drift withdraw` moves one to a withdrawn list, and the agent's `accept` then refuses it.
- **Scope `full` puts the parent in charge.**
  The agent's acceptances are `unruled` until the parent resumes from a decision halt.
  Every `Drift-Verify` halt but an error or transient stop is a decision halt, so the arbiter never settles one.
  `shedadapters.DecisionHaltPrefix` now covers these halts, and its doc says so.
- **Giving regression up is explicit.**
  `lyx drift defer` at a `Drift-Await` halt, or a `Drift-Baseline` halt on a failed or refused capture, and `lyx drift waive` at a `Drift-Verify` halt with no counted capture.
  The flag is read only when the run enters a Drift row, so turning it off while a row waits or is halted skips nothing.
- **Windows.**
  The step and loop jobs admit breakaway, the capture holds a named job through `proc.HoldNamedJob`, and `pairteardown` ends it through `proc.TerminateNamedJob`.
  When a job lyx did not create still refuses breakaway, loom's capture falls back to the foreground like darn's.
- **Out of scope.**
  A drift report format in `contracts/specs/`, the sandbox hub's harness (board note `csharp-sandbox-hub`), the Models-side config and skip-file reader (OSL-1046), and #609's other wants (drift outcome in the change description, per-batch snapshots, the slow-suite tier).

## To check by hand

- No test ran the real Windows paths: the gate runs Linux, so breakaway, the named job and its teardown are pinned by their seams only.
- Nothing ran against Models or a real .NET harness.
  Whether the harness reads `NORCE_BASELINE_DIR` in verify mode, whether its all-`Inconclusive` capture exits 0, whether its compare fails when it discovers no test, and that it reads no skip file yet, are for OSL-1046.
- In a .NET hub a Plan reviewer's build shares `bin` and `obj` with the capture's `dotnet test` in one worktree; `csharp-sandbox-hub` proves whether that breaks a capture.
- Review findings left open, to check in the code:
  - The dirty-tree check in `Drift-Verify` must read only the code checkout's tracked paths, never the records checkout (`fabricengine.SourceDirtyPaths`), so `_lyx/drift/scope.json` never counts as dirt.
  - A `Drift-Verify` halt must stay in place until its gate has run, not be cleared before.
  - The resume-time comparison against the last Stuck's `LastAt` or `At` must use matching time formats, or a resume from a decision halt is misread.
  - The compare marker's take must retry like the capture lock's, so a reader's probe cannot refuse a compare; the last commits retry it, so confirm the fix.
- The failing plan-level verify on `internal/shedbuild` was fixed by giving `DriftBaseline` and `DriftVerify` minimal config in the every-engine build test (`b5ac95755`); check that this config fits what the producers require.
