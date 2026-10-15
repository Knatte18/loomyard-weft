# Round 2 fixer report — discussion

The review's line numbers ran ahead of the file; each finding was located by its quoted text.

## Changes

- F1 (BLOCKING), fixed as proposed, prompt-only branch.
  Premise confirmed: `internal/loomcli/tacklerprompt.go:88` names only the operator-prompt deny for the autonomous posture, and the deny table documented in `internal/shuttleengine/claudeengine/doc.go` covers no Edit, Write, `git add` or `git commit`.
  D4's write-bound sentence and its Bound line no longer name shuttle's denies; both say the stencil's write rule is the only bound and is not enforced, and D4 states that no mechanical write check is added because the prime's other writers leave no baseline to compare against.
  Open risks gains the residual: a batten tackler's stray files, index changes or commit in the prime can be carried by a later Land into the push and deploy.
  The mechanical branch (refusing a resume into Land or Worktree-Teardown) was not taken: a tackler commit made during a stop at an earlier row, such as Run-Shed, reaches Land through the run's own progress, so that refusal would bound nothing it claims to.
- F2 (MEDIUM), fixed as proposed.
  D3 now names the records `shedrun` already declares with unchanged paths (drive reports via `DriveReportsDir`, driver lock, park marker, resume report, tackler directory, attempt directories and marker) and the settle records as the only record that changes home, from `SettleRecordsFile` over `LoomFrictionDir` to a per-run `shedrun` path, with its readers (the driver, `internal/arbiterstop`'s hold reading, batten's publish retry).
  The driver log stays a told path at `LoomDriverLog`, since `lyx loom start`'s handshake names it.
  The premise sentence now covers settle records only.
  Following from it, D8 and its acceptance line now say only the settle-record read moves; the tackler-marker and driver-lock reads already use `shedrun` (`internal/battencli/wire.go:1073`, `:1133`).
- F3 (LOW), fixed as proposed.
  D4's old-build sentence now says a new-build `lyx batten run` meets the old process's run lock as a busy stop and exits, and a rerun after that process exited resumes at the recorded row.
  The driver-lock acceptance line gains the busy exit.
- F4 (LOW), fixed as proposed.
  D5's migration replaces the comments of all four keys with the new template's, noting reconcile never refreshes an existing key's comment; the migration acceptance criterion checks all four comments.

## Disputed

None.

## Deferred

None.

## Process note

One read-only `git log` was run against `_lyx/discussion/decision-record.md` while orienting, before the no-git rule in the fix step was read; it changed nothing.
