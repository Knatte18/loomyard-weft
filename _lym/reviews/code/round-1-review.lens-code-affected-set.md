```yaml
lens: code-affected-set
outcome: reported
```

Per change, the set found:

- `doneSlugRefusal` became a package function: callers `checkRunStatus`, `battenPreStepUnmarked`, `DobbyTasks`; all updated. Covered.
- `TaskView.StartRefusal`: set only in `battencli.DobbyTasks` (sole non-test caller `beamcli/dobbydeps.go:150`); readers `heldByEntry`, `readyToStart`, `startReady`, `BuildView`; `doc.go` updated. Covered.
- `WaitingTask.StartRefusal` and `Ready()`: new JSON key `start_refusal` with `omitempty`; consumers `DobbyLines` and `beamcli` status; `battencli/cli.go` package doc updated. Not covered: the `beam status` help text, see F1.
- `heldStart` and the notice text: same `recordRefusal` path as a refused start; no stencil change forced. Covered.
- `PATTERN.md`: `PATTERN-dobby-starts-runs` not contradicted; no `Pattern-Departure:` trailer, no declared PATTERN changes. Nothing forced.
- Comment conventions: new doc comments self-contained, no end-of-line comments. No finding.

Findings

F1 — LOW, consistency — `internal/beamcli/cli.go:73-75`
- The `lym beam status` help text lists what a task waits on ("open dependencies, another machine's ownership, a pause, a refused start and its retry, or max_runs") and was not updated for `a start batten refuses: <text>` and `ready to start, last refused start: <text>`.
- `battencli/cli.go:27` was updated, so the two disagree.
- Fix: add "a start batten refuses" to the list and say a ready task shows its last refused start.

Tool calls (4):
1. Repo-wide grep for `ready to start`, `doneSlugRefusal`, `WaitingTask`, `start_refusal`, `waits on`: find readers and docs outside the diff.
2. Repo-wide grep for `refused start`, `could not start the task`, `TaskView`: find callers, stencil and help text.
3. `beamcli/cli.go:66-80`, `beamcli/doc.go:99`, orch stencil line 50, `DobbyLines`, `Ready()` callers: the manifest lacked them.
4. Grep of `dobbyengine` and `beamcli` docs for waiting-task prose.
