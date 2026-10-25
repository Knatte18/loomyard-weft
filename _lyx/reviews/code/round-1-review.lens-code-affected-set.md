```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — what the diff's changes force outside the lines it touches.

**Per change, the set found**
- **`render.Display.Title` and `overviewWindowTitle` (window naming).**
  - Set: the reed docs, the `reedengine` spawn path, the `orchcli/runs.go` caller, and the loom status strand.
  - Covered. The loom status strand (`loomcli/bootstrap.go:25`, `loomcli/doc.go:21`) sets no Title, so it keeps the name `overview`, which the decision record's Out list asks for.
  - Covered. The remaining "overview window" mentions in `reedengine` (`bar.go`, `follow.go`, `windoworder.go:1`, `ownwindow.go`, `doc.go`) name the anchor kind or the Title-less default, not the runs strand's window, so none is stale.
- **`DobbyLines` signature (now takes `runCells`).**
  - Set: `beamcli/cli.go:110`, `overviewdobby.go:80`, `overviewdobby_test.go:93`.
  - Covered. No other caller exists.
- **`lyx beam status` output (cells now include time in stage).**
  - Set: `beamcli/cli.go` and `beamcli/doc.go`.
  - Covered. `docs/overview.md` line 442 is updated.
- **Frame shape (no run list; dobby section, blank line, orch lines).**
  - Set: `battencli/cli.go` module comment, `docs/overview.md` batten row, `orchengine/doc.go`, `docs/overview.md` orch row, `orchcli/runs.go`.
  - Covered by those edits. One contradiction is left in `orchengine/doc.go` (finding F1).
- **Orch role stencil.**
  - The decision record lists it as a doc to update "where it names the window".
  - My grep of `contracts/` and `plugins/` found no stencil that names the overview window or the runs strand, so there is nothing to update.
  - Covered.
- **`docs/overview.md` after the Parent-Align merge.**
  - The orch, batten and beam rows keep the beam-window wording, and none of the old wording is reintroduced.
  - The parent's shortcut set (`a`, `prime`, `p`, `r`) in the shortcutcli row and the shortcuts row is kept.
  - The `lyx start` alias line is gone, and none remains (grep found none).
  - No task hunk is lost or duplicated.
  - Covered.
- **Parent's removal of shortcuts (`attach`, `pause`, `resume`, `start`) against the branch's added comments.**
  - No added comment in the branch's diff names `lyx attach`, `lyx pause`, `lyx resume` or `lyx start`, or a window order the parent changed.
  - Covered.

**Findings**

F1 — LOW, consistency — `internal/orchengine/doc.go:10` vs `:13`
- Line 10 still says `lyx orch` "runs from the hub's prime worktree only; every verb refuses elsewhere".
- The branch's rewritten `start` bullet on line 13 says the strand is named `beam: <slug>` "in a beam".
- `docs/overview.md` line 408 and `orchcli/runs.go` say `lyx orch` runs in the prime and in any beam.
- The task makes the doc speak of a beam, so the bullet and its header now contradict each other.
- Fix: change line 10 to say the verbs run from the prime or from a beam.
- Evidence: the `sed -n 8,14p` output of `internal/orchengine/doc.go`.

No other findings.

**Tool calls**
1. A repo-wide grep for "overview window", `DobbyLines`, `DobbyRunCells` and `NewDobbyRunCells`, to find the callers, tests and docs outside the manifest.
2. A grep of `contracts/` and `plugins/` for stencils naming the overview window, plus `sed -n 8,14p` of `orchengine/doc.go`, because the manifest lacked that header line.
3. A narrower grep of `contracts/`, `plugins/` and `.claude/` for "overview window" and "batten overview", to confirm no orch stencil names the window.
