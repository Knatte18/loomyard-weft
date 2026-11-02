```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness, judged on the changed functions and branches. I found no defects.

**List: function or branch, case run, result**
- `ensureRunsStrand` loop. With `[loom-status, runs]` the loom-status entry fills `loomStatus` and the later runs entry still fills `runs`. With `[runs, runs]` the second entry matches neither case, so the first is kept. The strand is picked correctly.
- `ensureRunsStrand`, a current `runs` strand. It returns `""` before the sidecar write, and the old code also returned before it. Correct.
- `ensureRunsStrand`, `runs` present but stale or dead. It replaces `runs` and ignores a co-listed `loom-status`, as the old code did. Correct.
- `ensureRunsStrand`, no `runs` and a `loom-status` strand. `ReplaceStrand(loomStatus.GUID, overview spec)` runs.
  - `replace.go:41` says replacing the overview strand itself is never refused.
  - If `loom-status` sits in its own window, `replace.go:41` still allows the replacement. The summary names this as a design decision.
  - The replaced strand's sidecar GUID goes stale. `ensureStatusStrand` then finds no matching strand and adds a new one beside `runs` in its own window.
  - Correct.
- `ensureRunsStrand`, no strand at all (the add path). The add runs, and a failure becomes a warning. Correct.
- `ensureRunsStrand`, error paths for `currentBuild`, `executable`, `Strands()`, replace, add and `WriteSidecar`. Each returns a warning through `runsStrandWarning`, which also logs it. `start` still succeeds. Correct.
- `runsStrandWarning`. It logs and formats once. The text matches the test's `wantWarning`. Correct.
- `beamUp`, `start.go:483`. The warning is appended to `report.Warnings` before `startOrch` appends its own, and the orch is launched whatever the warning says. Correct.
- `statusStrandAddSpec`, `nil` or other strands. It returns the overview display, as before. Correct.
- `statusStrandAddSpec`, a `runs` strand present. It returns own-window with `KeepViewer`. `validateDisplay` (`strand.go:70`) routes own-window to `render.ValidateDisplay`, which can only reject the Window/WindowSize fields, and both are zero here. Correct.
- `ensureStatusStrand`, the replace branch with `runs` present. It replaces a stale own-window `loom-status` with an own-window spec. `refuseWindowMove` applies to update, not replace, so there is no refusal. Correct.
- `ensureStatusStrand`, `statusResult.Strands` read at line 261 and used at 279. Nothing in between changes the strands, since the bootstrap lock is held. Correct.
- `pawlDeps` Beam-Up step. A `BeamUp` error returns the error as before. The success path returns `nil` after `surfaceBeamUpWarnings`, so it still returns `nil`. Correct.
- `surfaceBeamUpWarnings`, empty warnings. It returns before logging or posting. Correct.
- `surfaceBeamUpWarnings`, `NotifyOrchAt` returning an error. The error is logged and swallowed, so the row is not halted. Correct.
- `surfaceBeamUpWarnings`, `NotifyOrchAt` returning `posted=false` with no error (no orch recorded). The warnings are only logged, which the doc comment's "already in the log" covers. Correct.
- `surfaceBeamUpWarnings`, destination. `NotifyOrchAt(location, "PAWL", ...)` posts to the beam's own orch, the same as the existing `Notify` seam at `pawlwiring.go:136`. Correct.
- `beamUpWarningLine`, nil and multiple warnings. It returns `""` for nil, and for several it joins them with `"; "`, as the test pins. Correct.

**Findings**
- None.

Out of lens, noted only: on a `BeamUp` error the report's warnings are never surfaced. That is unchanged from before and is a design choice.

**Tool calls**
1. Read `internal/reedengine/strand.go:55-115`. The manifest lacked `validateDisplay`, needed to check that an own-window display with `KeepViewer` is accepted.
2. Read `internal/orchcli/notify.go:50-110`. The manifest lacked `NotifyOrchAt` and `postNotice`, needed for the `posted` and error semantics.
