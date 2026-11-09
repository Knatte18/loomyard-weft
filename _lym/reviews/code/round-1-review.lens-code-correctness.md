```yaml
lens: code-correctness
outcome: reported
```

**Task:** code-correctness lens over the perch-mode diff. No finding of BLOCKING or MEDIUM severity. One NIT.

**Per function or branch: case run, result**

- `freshRaddleEntry`:
  - Input: history ending `{Raddle-Write, goto}`. Result: fresh, position `len`. Correct. `shedengine/goto.go:140` and `:263` write `Producer: req.Target` with `OutcomeGoto` and no `RoutedTo`, which is what the new branch matches.
  - Input: `{Raddle-File, Stuck, RoutedTo: Raddle-Write}`. Result: fresh. Correct.
  - Input: `{Raddle-File, Stuck, RoutedTo: elsewhere}`. Result: not fresh. Correct.
  - Input: `{Record-Write, goto}`. Result: not fresh. Correct.
  - Input: empty history. Result: `false, 0`. Correct.
  - Input: Raddle-Write `Done` after a goto. Result: not fresh. Correct.
- `raddleWrite.firstMode`:
  - Input: `SplitPerch` true. Result: perch, and `AutoApprove` is never called. Correct.
  - Input: `SplitPerch` false with the key on or off. Result: auto or interactive. Correct.
  - Input: `SplitPerch` returns an error. Result: `p.fail` is returned before any state write. Correct.
  - Input: `AutoApprove` returns an error. Result: same. Correct.
  - Input: later calls with a recorded mode. Result: `firstMode` is not called, so `raddle_auto_approve` is not read again. Correct.
- `clearLastAttempt`:
  - Input: perch or auto mode. Result: the proposal is renamed to the returned path. Correct.
  - Input: the proposal file is missing. Result: `ErrNotExist` is tolerated. Correct.
  - Input: interactive mode. Result: only the approval record is removed. Unchanged.
- `raddleWriteEnv` and `HoldsSplitPerch(recipe)`:
  - Input: empty recipe name. Result: reads as tapestry, so false. Correct.
  - Input: a name that is not embedded. Result: error, and Raddle-Write fails on its first call. Acceptable, because loom runs load an embedded recipe at start.
- `RowsHoldSplitPerch`:
  - Input: a non-Bouncer row. Result: skipped. Correct.
  - Input: a Bouncer row with no `artifact_paths`, where the type assertion yields a nil slice. Result: false. Correct.
  - Input: a non-string path entry. Result: `text` is "", which does not match. Correct.
  - Input: a Bouncer over the proposal path. Result: true. Correct.
- `raddleBoardContext`:
  - Input: no state file. Result: `state.ReadJSON` yields the zero state, so `Perch` is false and the state read raises no error. This matches the new doc comment.
  - Input: state read error. Result: wrapped and returned. Correct.
  - Input: mode perch. Result: `Perch` is true. Correct.
- `raddleProposeVerb`:
  - Input: perch mode. Result: `needsNotice` is false, the note names the perch, `ProposedSession` is set, and the cold proposal is stored. Correct.
  - Input: auto or interactive mode. Result: unchanged.
- `raddleApproveVerb`:
  - Input: perch mode, with the run at the Raddle-Write row. Result: refused with a `way forward:` clause. The refusal fires before the proposed-session check, so it holds whatever the session state. Correct.
- `raddlePerchOutcome` and `slugs` in `internal/raddleparser/check.go`:
  - Input: perch off. Result: `perchOutcome` returns nil and `slugs` is unchanged. Correct.
  - Input: tapestry, or tasks with one part. Result: no finding. Correct.
  - Input: tasks with two parts. Result: `perch-outcome`. Correct.
  - Input: beam, project under another slug, project slug equal to the entry. Result: `perch-outcome`, `perch-outcome`, and no finding respectively. Correct.
  - Input: shape project with a nil project. Result: `perchOutcome` returns nil and `shape-invalid` reports it. No double report. Correct.
  - Input: `i == 0` is the project slug when a project exists, per `filedSlugs`. When there is no project, `perchProject()` is false, so the exemption cannot hit a part. Correct.
  - Input: a part under the Sketch's slug in a perch project. Result: still `slug-sketch`, plus `slug-duplicate` against the project slug. Correct.
  - Input: project slug equal to the entry. Result: `takenSlug` exempts it, since the entry is the exempt slug. Correct.
- `perchFilingRefusal` and its three seams in `decision.go`:
  - Input: the run has no RaddleFile row, or the current row is another row, or the run is not found so the current row is "". Result: returns ("", nil). `raddlePerch` and `raddleFiled` are not called, so a non-raddle recipe never reads raddle state. Correct.
  - Input: a raddle run outside perch mode at Raddle-File. Result: returns ("", nil), so the verb appends as before. Correct.
  - Input: perch at Raddle-File, `raddleFiled` reports not filed. Result: the goto to Raddle-Write refusal. Correct.
  - Input: perch at Raddle-File, filed. Result: the refusal lists the slugs. Correct.
  - Input: any seam returns an error. Result: `refuse("read the run's raddle filing: …; %s", decisionWayRetry)`. Correct.
- `raddleRunFiling`:
  - Input: no store entry for the slug, or a store entry whose `Sketch` differs. Result: `nil, false, nil`. Correct.
  - Input: a store entry whose `Created` differs from the board entry's. Result: not the run's, so false. Correct.
  - Input: a store entry and the task is gone from the board, which a one-task filing ends through `EndRaddleEntry`. Result: `found` is false, so it returns `held.Filed, true`. Correct.
  - Input: a board read error. Result: wrapped and returned. Correct.
- `raddleModeRules`, `raddleReturnNote`, `RaddleTable` and `entries_raddle`'s `Table(recorded.Mode, …)`:
  - Input: perch mode. Result: the perch text is rendered, the output is the proposal and `Interactive` is false. Correct.
  - Input: mode "" (not yet recorded). Result: falls to the auto text, as `interactive=false` did before. Correct.
  - Input: return note in perch mode. Result: set-aside wording, matching the proposal rename. Correct.

**Findings**

1. NIT, `consistency`, `internal/loomrecipe/recipekeys.go` in `RowsHoldSplitPerch`. It compares `filepath.FromSlash(text)` to the proposal path with no `filepath.Clean`. A recipe row naming `./_lym/raddle/proposal.yaml` is silently not read as a split perch, so the run falls back to auto or interactive mode. That fallback is the safer direction, so this is only a precision gap. Fix: compare `filepath.Clean(filepath.FromSlash(text))`.

Tool calls (3):
1. Read `ReadRaddleState`, `filedSlugs` and `takenSlug`: my context had only the diff hunks, and I needed the missing-file behaviour, the project-slug ordering and the slug-taken exemption.
2. Grep for `OutcomeGoto` in `shedengine`: to confirm what `Goto` and `Relaunch` write into history.
3. Grep for `raddleBoardContext(` callers: to confirm the new state lock read does not run while a state lock is held.
