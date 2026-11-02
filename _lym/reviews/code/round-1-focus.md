---
round: 1
focus:
  - "internal/orchcli/runs.go `ensureRunsStrand`: when no `runs` strand exists and a `loom-status` strand does, `ReplaceStrand` takes the loom-status GUID. Settle whether the replaced strand's window, viewer and sidecar state stay consistent, and whether the prime's runs strand (not a beam) can wrongly replace a `loom-status` strand."
  - "internal/orchcli/runs.go: the rewritten loop matches `runs` and `loom-status` in one `switch`. Settle whether a `runs` entry that follows a `loom-status` entry, or two entries of one role, is still picked correctly, and whether the early return for a current `runs` strand skips the sidecar write the old code also skipped."
  - "internal/orchcli/start.go and every other caller of `ensureRunsStrand` (grep for it): settle whether each caller now handles the returned warning, or drops it as `BeamUp` used to drop its report."
  - "internal/loomcli/bootstrap.go `statusStrandAddSpec` and its callers: settle whether every call site passes the session's strands read at the right moment, whether the own-window `KeepViewer` display is one reed accepts beside an overview strand, and whether the re-entrant `ensureStatusStrand` keep/replace path still finds a `loom-status` strand it placed in an own window."
  - "internal/loomcli/pawlwiring.go `surfaceBeamUpWarnings`: settle that a failed `NotifyOrchAt` or a warning-bearing report never halts the Beam-Up row, that the notice reaches the beam's orch (right location and program), and that the success path of `BeamUp` still returns `nil` as before."
  - "Scope against the decision record's Goal and its Run instructions: settle whether the commit message and `_lym/landing/summary.md` bear out that candidate 2 was confirmed from the code (batten's child-run bootstrap running `lym loom start` in the beam's own session), since fix 1 licenses only on that confirmation; and whether the exported `RunsStrandName` and `loomengine.LoomStatusStrandName` renames leave any old literal behind."
  - "Tests: settle whether each behavior the diff adds has coverage (the replace-loom-status row, the own-window status add beside a runs strand, the warning on the report, the notice text) and whether any added test duplicates an existing one."
  - "Repository rules: read the repository's constraints document and the docs it names for this kind of change; settle whether the `doc.go` edits in `internal/loomcli` and `internal/orchcli` are the docs required and whether the new comments follow its comment conventions."
---

Round 1 seed. The diff is one commit plus a comment-break follow-up, so the focus follows the three changes in the commit message: the runs strand replacing `loom-status`, the add-side rule in `lym loom start`, and Beam-Up surfacing the report.
