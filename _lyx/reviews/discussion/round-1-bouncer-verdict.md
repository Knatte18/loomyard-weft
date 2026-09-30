---
verdict: APPROVED
rationale: "All eight round-1 findings (F1-F8) are fixed in the current pair: the once-marker gate is 'no marker present' at both spawn sites with a fresh-child clear and a lost-marker rule, the shared info/exclude write is a lock-and-atomic-replace Constraint with a subpath-anchored AC, the tracked tasks.json case is carved out with its own AC, and the folder Decision states the session-targeting requirement with a CodeLauncher AC; no new deficiency found."
---

# Discussion-Review round 1 — judge verdict

The reviewer returned `APPROVED` with three MEDIUM, three LOW and two NIT findings.
The fixer's `round-1-fixer-report.md` claims every finding fixed; I checked each against the current `decision-record.md` and `support-log.md` rather than trusting the report.

- **F1 (once-per-run gate):** the Decision now states the gate as "no once-marker present" at both `Call` and `callAwaiting`, written after the attempt whatever its outcome, cleared when `Call` finds no child status file.
  The resumed-run Open risk now matches that reading, and new criteria cover the `callAwaiting` re-spawn and a leftover marker from an earlier run.
  This agrees with `innerrun.go`, where `Call` spawns on `!found` and already clears the done-seen marker the same way.
- **F2 (shared exclude file):** the new Constraint resolves the file with `git rev-parse --git-path info/exclude`, writes under `fabricengine`'s existing exclude lock (`mutateGitExclude`, `exclude.lyx.lock`) with an atomic replace, and requires a pattern matching `.vscode/` at the anchor subpath.
  The Decision says "the repository's shared `info/exclude`", and the criteria cover a subpath-anchored hub and line preservation.
- **F3 (tracked `tasks.json`):** the Decision overwrites only an untracked `tasks.json`, leaves a tracked one untouched with a warning, and still opens; a matching criterion and Q12 exist.
- **F4 (lost scratch):** the Decision states that a lost marker re-opens on the next successful spawn, and that this is intended.
- **F5 and F6 (folder):** the folder Decision now requires the opened folder to be the bootstrap's directory, and a criterion pins the stubbed `CodeLauncher` argument to `<hub>/<slug>/<AnchorRel>`.
- **F7 and F8:** markers are named by `SpawnConfirmedFile`, and the illustrative signature and re-argued split-row clause are gone from the closure Decision.

The report has no `## Focus departures` section, so there is nothing to ratify or reject.
Its "Focus verdicts not covered by a finding" section is sound: no support-log content carries a requirement the plan writer lacks.

My own pass over the revised pair found nothing to add.
The Decisions' remaining "not X" phrasing (`not Seed-Child`, `not a new recipe row`) states the decision boundary, not a rejected-alternative narrative, and the alternatives themselves sit in the support log.
The support log's `## Review rounds` still reads `_No rounds yet._`, but the stencil assigns that section to the gate, not the writer, so it is not a finding.
