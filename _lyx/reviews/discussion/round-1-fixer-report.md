# Fixer report — discussion round 1

Every finding in `round-1-review.md` was fixed; `lyx loom validate-discussion` passes on the edited pair.

## Changes

- **F1 (once-per-run gate):** In `decision-record.md`, the once-per-run Decision now defines the gate as "no once-marker present" at both spawn sites (`Call` and `callAwaiting`).
  `Call` clears the marker when it finds no child status file, so a leftover from an earlier run of the slug cannot suppress the open.
  The Scope bullet no longer says "first", and the resumed-run Open risk is rewritten to match the gate.
  Two acceptance criteria were added: an open on an approval re-spawn when no marker exists, and a leftover marker from an earlier run that does not suppress the open.
- **F2 (shared exclude file):** Added a "Shared exclude file" Constraint: resolve the file with `git rev-parse --git-path info/exclude`, write it under `fabricengine`'s exclude lock with an atomic replace, and use an entry that matches `.vscode/` at the anchor subpath.
  The Decision now says "the repository's shared `info/exclude`", and the Open risk was trimmed to the repo-wide effect.
  The clean-tree criterion now covers a subpath-anchored hub, and a new criterion covers the lock and preservation of existing lines.
  The Note now points at `gitexclude.go` and warns against copying its root-anchored patterns.
- **F3 (tracked `tasks.json`):** The `tasks.json` Decision now overwrites only an untracked file.
  A tracked one is left untouched with a warning, and the open still happens.
  The overwrite criterion now names the untracked case, a new criterion covers the tracked case, and the Auto-mode assumption says "untracked".
- **F4 (lost scratch):** The once-per-run Decision now states that a lost marker re-opens the window on the next successful spawn, and that this is intended.
- **F5 (attach premise):** The folder Decision now requires the opened folder to be the directory the bootstrap ran in, so `reed attach` resolves the child's own session.
- **F6 (folder criterion):** Added a criterion that the stubbed `ideengine.CodeLauncher` receives `<hub>/<slug>/<AnchorRel>`.
- **F7 (marker name):** The markers are now referred to as `SpawnConfirmedFile`, approval-acted and done-seen.
- **F8 (deliberation in a Decision):** Dropped the illustrative closure signature and the rationale clause that re-argued the split-row alternative.
- `support-log.md`: the new picks are recorded as Q11–Q13 in the Question ledger.

## Deferred

None.
