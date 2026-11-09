```yaml
lens: existing-paths
outcome: reported
```

Task: the existing-paths lens. For each decision, trace the existing code paths that write, reset, reapply or read the state it changes.

**Paths traced, per decision**

- D1 worker under `runproc.HubOwner`:
  - `lym fabric anchor`'s `EndDobbies` on the prime ends every role but push and sync: **breaks** (finding 3).
  - `orchcli` `EndRole(RoleDobby)`, `pairteardown` `EndSparing(slug)` and slack `EndRole(RoleSlackRelay)`: survives.
- D2 `adopt.json` and `adopt.lock` beside the subject directories:
  - `ReadCheckViews` skips anything that is not a directory: survives.
  - Per-subject `holds.json` and `fix-tries.json`, written through `deps.go:108-110`: survives. D5 drops holds; tries stay.
- D4 suppression:
  - Orch watcher `ownerTick` (`watcher.go:279`): survives if it reads the record first.
  - Dobby loop `beamOwnedElsewhere` exit (`loop.go:494`): survives.
  - A pass already in flight exits owned elsewhere, then `ownedElsewhere` gives it an `OwnerCaller` pause (`owned.go:40`), which the `pause` step turns into an adopt entry: survives.
  - `recheckOwnerPause` lifts `OwnerCaller` entries only (`owned.go:104`, `dobbydeps.go:494`): survives.
- D5 `AdoptCaller` entry:
  - `tenderengine.Pause` answers `AlreadyPaused` on any live entry of the scope (`pause.go:115-118`): survives.
  - Hibernate wake `holdRuns` pauses through tender, and the existing entry wins: survives.
  - Hibernate `watch.go:184` lifts only the request's own entry: survives.
  - Beam start's Prepare `pauseOwned` (`orchcli/doc.go:208`) adds nothing over a live entry: survives.
  - `CoverageOf` sets `Held` for `HoldCaller` only (`record.go:229-240`), and `planPass` gates on `Held` (`passes.go:120`). This survives only if `Held` widens to `AdoptCaller`; D5 says the dobby treats the entry as it treats the hold, so the widening is stated.
  - Run report `holdFound` on `HoldCaller` (`runreport.go:139`): replaced by `paused_by_adopt`, stated.
  - After `lym tender resume`, `recheckOwnerPause` lifts no entry but sets `liftPending`, so the run gets its full pass: survives.
- D7 `BeamUp` without Prepare from adopt: survives.
- D8 green end:
  - `liftHolds`, `startLiftedRuns`, `BeamUp` and `resumeHaltedChildren` are dropped: survives.
  - `checkSubjects` read `heldSubjects` from `holds.json`; D3 now keys on the owner write: survives, stated.
- D10 `lym s`: `runArgvs` stops at the first non-zero exit (`shortcut.go:218`): survives.
- D11 archive tags under `archive/&lt;slug&gt;/`:
  - `cleanupremotewarp.go:239` and `remoteleftover.go:224` read tags by slug prefix: not traced further.
  - `reportengine` (`runs.go:139`, `cost.go:136`, `timing.go:242`, `raddle.go:71`) and `recordsreads.go:56` read archive tags as run records: **breaks / unstated** (finding 1).
- D12 `--archives` reads the `archive/&lt;slug&gt;/` prefix, which teardown and `--drop-local` also write: **misreads** (finding 2).

**Findings**

1. MEDIUM · design · decision-record.md D11, step 2.
   D11 pushes archive tags for runs that are still live, now driven on the new owner, and for the `root` and beam slugs. Today `archive/&lt;slug&gt;/` tags are written only for runs being torn down or dropped.
   Existing readers treat every archive tag as a record of a run's archived tip: `internal/reportengine/runs.go:139`, `cost.go:136`, `timing.go:242`, `raddle.go:71` (all `ArchiveTagSlug`) and `internal/fabricengine/recordsreads.go:56` (a per-slug archive-tag read).
   The record is silent on whether these readers keep, skip or misread the new tags. A live run could be reported as archived or ended, and a `root`/beam slug could appear as a run.
   Evidence: the grep results at those lines. I did not read their bodies, so the exact misreading is unconfirmed.
2. LOW · design · decision-record.md D12.
   `--archives` lists every `archive/&lt;slug&gt;/&lt;sha12&gt;` tag whose commit the slug's origin branch lacks. The same prefix holds the teardown's archives and `--drop-local`'s archives, which the adopting machine itself writes.
   A torn-down run's tags always qualify, because its origin branch is deleted. So the list mixes those with the old owner's tags and calls them all the old owner's archives.
   Evidence: `internal/fabricengine/archive.go:78` `ArchiveTagName`, and `cleanup.go:81-85` (teardown and cleanup write the same tags).
3. LOW · design · decision-record.md D1.
   The worker registers under `runproc.HubOwner`. `lym fabric anchor` ends the prime's dobby through `EndSparing(runProcDir, HubOwner, pid, RolePush, RoleSync)`, which ends every other `HubOwner` role. That would kill a running adopt worker mid-step, possibly during `deploy` or `owner`.
   D2's resume recovers, but D1 states the worker outlives its caller without naming this ender. The fix is to spare the adopt role, or to state the bound.
   Evidence: `internal/fabriccli/anchordeps.go:91-96` and `:258`, `internal/fabriccli/anchor.go:468`, `internal/runproc/end.go:67`.

**Tool calls**

1. `runproc` End/EndRole/EndSparing callers: which paths end `HubOwner` processes; not in the manifest.
2. `checkview.go` 30-80 and the holds/run-state users: whether `adopt.json` beside the subject directories is misread, and who reads `holds.json`.
3. `orchcli` Prepare/owner-pause grep: beam start's owner-pause path that `lym s` runs after adopt.
4. `tendercli/deps.go` 100-120, `anchordeps.go` spans and `runproc/end.go` 55-80: the bodies of the two enders of hub-owned processes.
5. `EndDobbies` callers: which roles the anchor spares.
6. Archive-tag prefix readers: who reads the tags D11 now writes.
