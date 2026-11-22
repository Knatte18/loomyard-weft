```yaml
lens: code-correctness
outcome: reported
```

Code-correctness lens: I ran each changed function and branch in the diff step by step with concrete inputs, including empty and failure cases.

**Functions and branches run (correct or not):**
- `configengine.TopLevelValue`: tried an empty or comment-only layer (nil map, reads unset), a key set to null (set, value nil, which `checkTaskSetting` then refuses) and a local file that fails validation (error marked ErrInvalid). Correct.
- `dobbyengine.lookupTaskSetting`: tried an empty chain (falls to the template), no `Board` (skips the hub default), and a lower anchor setting `off` over a parent's `on`. Correct.
- `LookupLandNotice`: on a lookup error it returns false with the error. Correct.
- `checkTaskSetting`: tried `on` written unquoted, which yaml.v3 decodes into `any` as the string "on" (passes), a nil value (refused) and an int (refused). Correct.
- `hubgeom.TaskSettingChain`: tried an empty slug (prime alone), a task whose worktree is absent (left out), a stat error (returned), a `ParentOf` error (returned) and a cycle (stops at the prime). Correct.
- `LoadConfig` per-task check: tried a beam worktree file holding `required`. It is refused, so that beam's dobby stops loading beam.yaml. This works as written; the summary declares it as an open risk.
- `ReinterpretPerTaskSettings`: tried an empty file (returned unchanged), a non-mapping document (unchanged), a non-scalar value (skipped) and `required` (rewritten to `"on"`). Correct, except the claim in its rewrite line (F3).
- `flagPrimeRetiredValues`: tried a prime local file that fails `ValidateLocal`. The error aborts the reinterpretation and the whole hub-wide reconcile. Not correct (F4).
- `daemon.readLandNoticeRequests` / `rememberRuns`: tried a lookup that fails on every tick while the run is ahead of Land, then the run lands. Requested stays false, no line is decided and the memory is dropped. Not correct (F1).
- `rememberRuns` with an old state record holding `{requested:true, clear_owed:true, line:""}` and the run still listed past Land: a second notice goes out. Not correct (F5).
- `rememberRuns` with a board read failure: membership keeps its last reading. Correct.
- `settleLandNotices` / `dropLandNotice`: tried a failed post (line kept) and a successful post (memory dropped). Correct.
- `foldLegacyBeamFields` / `setBeamSettingIfAbsent`: tried absent keys (nothing happens), a non-string `pull_request` (dropped), `notify_on_land: false` (dropped), a project (dropped), `beam` that is not a mapping (left alone) and a beam value already set (kept). Mechanically correct; the carried meaning of `off` is not (F2).
- `validateUpsertFields`: an upsert naming `pull_request` or `notify_on_land` is refused with a hint. Correct.
- `ResolvePullRequest`: tried `merge-in` on a non-beam with a GitHub rule (TrueMerge, no pull request), `merge-in` on a beam with none recorded (reads as off), `merge` with a local origin (halt), `on` with no upstream (no pull request), and an empty branch (no pull request). Correct.
- `resolveWithParentRule`, `judgeNoPullRequest`, `closeFollowUp`, `prClose`: tried `merge` versus a GitHub rule demand. Correct.
- `loomcli.parentPullRequestMode`: tried a beam landing (beam's own value first) and a `primeOfHub` failure. The halt says the pull_request "could not be read" and carries the prime error. Acceptable.
- `beamcli.hubLandDeps.PullRequestMode` and `dobbydeps.LandNotice`: tried a beam slug. The chain is right.
- `DobbyTasks`: a placement error for a released task fails the read. Correct.
- `BuildView`: `LandNotice` comes from the remembered reading. Correct.
- `Store.Prune`, `EndRaddleEntry`, `scopeChanged`: tried a done entry (pruned) and a raddle entry (status only). Correct.

**Findings**

- **F1.** MEDIUM, design, `internal/dobbyengine/landnotice.go:72-91,118-124`. A run whose `land_notice` lookup never reads while it is ahead of Land is treated as off, so its notice is lost when it lands. The memory is dropped after one log line.
  - Before this diff, a failed beam.yaml read set `decides` false and deferred the decision until the read came back.
  - Now `requests` simply leaves the slug unset, `Requested` stays false, and `rememberRuns` decides "no line" and forgets the run.
  - This is likely in practice: no `CheckValues` exists for the beam module (`configreg/runsettings.go:144-151`), so a run_settings value such as `land_notice: per-task` reaches the local layer and fails every lookup.
  - Evidence: `landnotice.go:84-88` (the failure only logs); `rememberRuns` (`if note.Requested || …`, otherwise the memory is deleted).

- **F2.** MEDIUM, design, `internal/boardengine/store.go:566-569` (fold) together with `reinterpret.go:322-326`. Carrying a legacy board `pull_request: "off"` verbatim changes what it means.
  - Old behaviour: a field `off` under a `required` or `merge` parent still opened a pull request; the old `ResolvePullRequest` only warned ("still required").
  - New behaviour: the task's own `beam.pull_request: off` overrides the parent's `merge`, or the `on` that `required` is rewritten to. The task then lands by direct push with no pull request; under a `merge` parent that push hits a branch the remote takes only through a pull request.
  - Evidence: the base `pullrequestmode.go` `required := parentMode == required || merge …; if required &amp;&amp; entryField == off { Warn … still required }`, against the new `lookupTaskSetting`, where the first anchor that sets the key wins.

- **F3.** MEDIUM, design, `internal/dobbyengine/reinterpret.go:26-28,388-391`. The `required`→`on` rewrite tells the orch the meaning is kept ("a landing still goes through a pull request where the origin allows one"), but it is not kept for beams.
  - Under the old `required` parent, `lym beam ready` refused a beam with no pull request recorded, and Finalize opened one.
  - Under `on`, a beam with none recorded wants none: `ResolvePullRequest` only wants one under `on` when `!isBeam` (`pullrequestmode.go:143`), and `judgeNoPullRequest` passes.
  - The flagged line in the mail is therefore false for every beam under such a hub. Evidence: the removed `land_test` row `RequiredParentWithNoneRecordedRefusedNamingOpen`; new `pullrequestmode.go:111-145`.

- **F4.** MEDIUM, design, `internal/dobbyengine/reinterpret.go:367-384` and `internal/hubreconcile/walk.go:240-243`. An advisory flag read turns into a fatal error for the whole reconcile.
  - `flagPrimeRetiredValues` reads the prime's files with `TopLevelValue`, which runs `ValidateLocal` on the local file. If the prime's `beam.local.yaml` is unreadable or holds a key that fails validation, `ReinterpretPerTaskSettings` returns an error.
  - `applyReinterpretations` then fails, and with it the hub-wide reconcile `Write` (and every command waiting on the binary-change reconcile).
  - The read only exists to flag. A failure there should become a flagged line, not abort the rewrite of the hub file.

- **F5.** LOW, design, `internal/dobbyengine/landnotice.go:23-34,118-124`. On upgrade, a run whose old memory owed only a clear gets a second landing notice.
  - The old binary blanked `Line` after a successful post and left `ClearOwed` set when the clear failed.
  - The new `LandNoticeState` has no `ClearOwed`, so such a memory reads `owed()==false`. While the run is still listed past Land, `rememberRuns` decides a fresh line because `Requested` is true, and the already-sent notice is posted again.
  - Evidence: base `settleLandNotices` (`updateLandNotice(... Line = "")`, then on a clear error `continue`); new `rememberRuns` `if !remembered || note.owed() || !decides`.

No other function I ran produced a wrong result, a swallowed or misreported error, or state left inconsistent on a failure path.

Tool calls:
1. `internal/hubreconcile/walk.go:215-262`: to see whether a reinterpretation error aborts the hub-wide reconcile Write (F4). My context held only `applyReinterpretations` itself, not its caller.
