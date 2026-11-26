```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each function and branch the diff adds or changes against a concrete case.

**Function and branch list**

- `dobbyengine.LookupWithSource`:
  - Chain `{Board only}`, no anchors: the board layer is at index 0, which is `&gt;= len(Anchors)` (0), so it answers `HubWideSource`. Correct.
  - Anchors `[task, parent, prime]` with parent set: answers index 1. Correct.
  - Nothing set anywhere: the template decides and it answers `HubWideSource`. Correct.
  - `ownTaskSetting` or expand fails: answers `"", 0, err`. The 0 is never read on the error path. Correct.
  - `checkTaskSetting` refuses the value: the error is returned with the index, and callers check the error first. Correct.
- `dobbyengine.lookupTaskSetting`: delegates and drops the index, so its answer is unchanged. Correct.
- `hubgeom.TaskSettingChain` `Names`: appended in lockstep with `Anchors` in both the worktree branch and the prime branch, so the lengths always match. With an absent worktree, both are skipped together. Correct.
- `hubgeom.BeamModes`:
  - Prime does not resolve: error. Hub-wide or root read fails: error. The render then warns and shows no mode line. Correct as documented.
  - One beam fails: warning, beam left out, other beams kept. Correct.
  - `beams` nil: empty map. Correct.
- `hubgeom.primeOfBoard`:
  - Bolt dir is a linked worktree of the records repo and `worktrees[0]` is the prime's weft sibling: `ResolveWithin` maps it to its owner. Correct.
  - Main worktree is not a weft sibling: `ResolveWithin` returns the records main itself, the `prime.WorktreePath() == recordsMain` guard fires, and it errors. Correct.
  - Records repo whose refs are not files: see finding 2.
- `hubgeom.readBeamMode`:
  - Beam worktree present, with origin and landing record: settings, parent branch and PR all filled. Correct.
  - Worktree absent: `ReadOriginFor` stat finds nothing, so `found=false` and the branch stays empty. `WorktreePath` stat gives `ErrNotExist`, so it returns with no PR. Correct.
  - `.weft` lock dir missing: error, beam left out with a warning. Correct.
  - Landing record names another slug: not found, no PR. Correct.
  - Landing record whose PR is merged or closed: see finding 1.
- `hubgeom.readBeamSettings`:
  - autonomy `"true"` maps to `"on"`, `"false"` to `"off"`. `checkTaskSetting` answers only `"true"` or `"false"`. Correct.
  - Either lookup fails: error. Correct.
- `hubgeom.readBeamSetting`:
  - anchor 0 with name `beam` gives `SetBy` `beam`. Correct.
  - Prime anchor (name `""`) gives `RootSubject`. Correct.
  - `HubWideSource` gives an empty `SetBy`. Correct.
- `boardengine.InstallBeamModeReader` / `installedReader`: `nil` stores a pointer to a nil func, so `installedReader` answers nil and `readBeamModes` short-circuits. Correct.
- `boardengine.readBeamModes`:
  - `out.BeamModes` nil: returns nil.
  - Filter: a done beam, a non-beam project and a task are all excluded.
  - Reader error: warning, nil.
  - All correct.
- `boardengine.Config.Outputs`: every `boardengine.New` site (`boardcli`, `retrocli`, `ideengine`, `serveopen`, `battencli`) builds inside a command, after the pre-run has installed the reader. The replay and owner-write paths call `cfg.Outputs()` at render time. Correct.
- `boardengine.modeHeader`: nil modes give nil. With modes, it gives four lines and a blank. Correct.
- `boardengine.modeLines`:
  - Root (`Slug ""`): uses `Root` with own = `RootSubject`. Correct.
  - Beam missing from the map, or `Beam` false: nil. Correct.
  - Root with every setting hub-wide and no parent or PR: parts are empty, so nil. Correct.
  - PR number 0: no link. Correct.
- `boardengine.everyPlaceHubWide`:
  - Root `SetBy` `""` and `Beams` empty: true. Correct.
  - Any beam `SetBy` non-empty: false. Correct.
- `boardengine.settingText` / `sourceText`:
  - `SetBy` equal to own: bold. Correct.
  - `SetBy` `""`: `(hub-wide)`. Correct.
  - `SetBy` `RootSubject`: `(P0)`. Correct.
  - `SetBy` is an intermediate non-beam pair on the board: its handle. Not on the board: the slug in a code span. Correct.
- `boardengine.RenderToDisk` / `renderFiles` / `Render`:
  - Modes are read before render, and a nil read leaves every file without a mode line.
  - `Render` passes nil.
  - Board write under the write lock: `ParentOf`'s `ReadTasks` takes only the store's swap lock, which no one holds after `Save`, so there is no deadlock.
  - The landing mutators (`beamcli` `RecordDescription` and `WriteLanding`) take no board lock while they hold the landing lock, so there is no lock-order inversion.
  - All correct.
- `boardengine.fileHeader` / `projectSectionContent`: the mode lines sit after the facts line, and a nil result adds nothing. Correct.

**Findings**

1. **MEDIUM**, design. `internal/hubgeom/beammodes.go` `readBeamMode`, at the `found &amp;&amp; record.PullRequest != nil` branch.
   - The board shows the beam's PR as "Beam PR [#N]" whenever the landing record names one, but the record has no state and stays set after the PR is merged on GitHub (until the beam's Finalize drops it) and after it is closed.
   - A closed PR halts Finalize without any drop, so the README keeps linking a closed PR as the beam's PR. The entry asks for the beam's *open* pull request.
   - Evidence:
     - `internal/pawlengine/landing.go:27-34`: `LandingPullRequest` has no state field.
     - `internal/landingshed/finalize_pr.go`, `recordedBeamLanding`: `StateClosed` returns `halt(...)` with no `dropBeamRecord`, and the drop runs only once the landing completed (`finalize.go:421-422`).
2. **LOW**, design. `internal/hubgeom/beammodes.go` `primeOfBoard` calls `dotgit.ListWorktrees` directly.
   - For a records repository whose refs are not files, it returns `ErrRefStorage`, so every render on such a hub drops all mode lines and logs a warning.
   - Fabric's lister handles that case through porcelain.
   - Evidence: `internal/dotgit/worktrees.go` (`ErrRefStorage` doc) and `internal/fabricengine/worktreelist.go:35-39`, where `List` falls back to `listFromPorcelain`. `hubgeom/autonomy.go`'s `resolvePrimeWithoutGit` shares the limitation.

Tool calls:
- `sed internal/beamcli/deps.go:305-335` and `grep boardengine.New(` callers, in one call. My context lacked whether the landing-record mutators take the board lock (lock order under the board write lock), and whether every `Board` is built after the root pre-run installs the reader.
