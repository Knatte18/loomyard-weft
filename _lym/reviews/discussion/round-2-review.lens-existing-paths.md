```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. Task: trace the existing code paths that write, reset or read what each decision introduces or changes.

**Paths traced, per decision**

- **D1, `d.listedRuns` as the watch's list.** `drivePasses` sets it on every scheduling tick (`passes.go:181`), and `clearDoneRuns` and `landNotices` read the same list (`loop.go:719`, `donerun.go:38`). Survives.
- **D1, child read gated on status mtimes.** `RunView.StatusModified` and `ChildStatusModified` already exist (`passes.go:38-39`). `triggerFor` keeps its own `childSeen` memory (`passes.go:344-351`), which the watch does not share. Survives.
- **D1, child status vanishing at Worktree-Teardown.** `ReadChild` returns found=false (`runnotice.go:107-108`), and D1 keeps the last child part. Survives.
- **D1, run on a machine that does not own it.** `DobbyRuns` lists runs whose owner is another machine and only marks them (`dobbyreads.go:160`, `owned.go:25`). Nothing in the watch's path skips them. **Breaks**, see F-A.
- **D2, `run_watch` in config.** It has to join the key loops in `LoadConfig` and `CheckLocalValues` (`config.go:130,146`). `configreg` wires `CheckLocalValues` as the beam module's `CheckValues` (`configreg.go:147`), so board run_settings and local-layer writes get the refusal. `boardengine`'s legacy fold names only `pull_request` and `land_notice` (`store.go:404`), which is migration of retired keys and does not concern a new key. Survives.
- **D2, `checkTaskSetting`.** It falls back to `LandNoticeValues` for any key other than `pull_request` (`config.go:169-172`). The record's "joins the per-task check" covers this. Survives.
- **D3, copying the `land_notice` reading.** `rememberRuns` writes `LandNotices` and drops it at PastLand (`landnotice.go:119-156`). The watch runs after `landNotices` in the tick (`loop.go:719`), so the copy is taken before the drop. Survives.
- **D3, a cancel that removes its records.** `clearRunPhase` deletes the records (`cancel.go:544-565`), `dobbyParentRuns` drops a run with no status (`dobbyreads.go:104-111`), and `postNotice` is tried once (`cancel.go:830-836`). The record states this bound. Survives.
- **D4, the watch map in the state record.** `recordOwed` deletes only `s.Runs` (`passes.go:333-337`), the tick's `UpdateState` touches PID and sleeps (`loop.go:678-696`), and `clearDoneRuns` deletes only `CountedLandings` (`donerun.go:72-77`). The map survives all three.
- **D4, the done-run clear.** It already skips runs that `LandNotices` remembers (`donerun.go:45`), and D4 adds a watch-owed skip beside it. Survives.
- **D4, dropping an entry when the batten status is gone.** The status-exists seam already exists as `Deps.StatusExists` (`dobbydeps.go`). A run left out only because its status is unreadable keeps its entry. Survives.
- **D4, a run that moves to another beam's parent.** `dobbyParentRuns` drops it once its parent is no longer this dobby's (`dobbyreads.go:101`), but its batten status still exists. Its watch entry is never dropped. This is a small leak, not a wrong line, so I raise no finding.
- **D5, posting through `NotifyRunFrom` with empty tags.** No run tag means a teardown drops nothing, and no topic means nothing is replaced (`notify.go:73`). Survives.
- **D5, "no orch strand recorded" waits without counting an attempt.** Every existing dobby post seam goes through `requirePosted`, which folds a no-strand result into an ordinary error (`dobbydeps.go:89-96`). **Misread** if the watch's seam reuses it, see F-B.
- **D6, the holder record's new marker field.** `tryAcquireSlot` writes it (`pool.go:316-332`). `Holders` reads it for `lym gate status` and the dobby (`pool.go:338-369`), decoding without `KnownFields`. Survives.
- **D6, task worktree resolution.** `dobbyWatchDirs` resolves the pair through `pairLocation(prime, slug)` (`dobbywatch.go:47`), which matches "as the dobby's file watch resolves it". Survives.
- **D6, which process carries the marker.** The marker is `LYM_GATE_TEST_PID=<pid>`, set in the same process that calls `AcquireSlots` (`run.go:172`, `run.go:822`). Survives.
- **D7, code that keys on a notice prefix.** Only `battenshed/notice.go:26` and `runnotice.go:24` define a prefix, and no routing code matches one, so `[run watch]` breaks no existing reader. Survives.

**Findings**

- **F-A — MEDIUM, design, `_lym/discussion/decision-record.md:42` (D1 run list) and `:182` (PATTERN-owner-machine constraint)**
  - Summary: the watch reads every listed run, but `DobbyRuns` also lists runs owned by another machine. So a non-owner machine's dobby of the same parent worktree posts its own watch lines from its local copy of the batten status as catch-ups pull it. The orch gets duplicate and late batten-level stop and landed lines. The record's constraint "the dobby runs on the owner machine" does not hold for this listing path, and the record never says whether the watch skips `RunView.OwnedElsewhere` or a run whose state record holds `OwnedElsewhere`.
  - Evidence:
    - `internal/battencli/dobbyreads.go:160`: "It carries whether the registry reads its owner as another machine."
    - `internal/dobbyengine/owned.go:25`: such runs are marked, not dropped.
    - `internal/dobbyengine/leftover.go:78` and `passes.go:141`: existing steps skip such runs themselves.
  - Fix: in D1, state that the watch reads no run whose reading marks it owned elsewhere (`OwnedElsewhere`, in the view or the state entry). Add an acceptance bullet: a run owned by another machine posts no watch line here.
- **F-B — LOW, design, `_lym/discussion/decision-record.md:122` (D5)**
  - Summary: D5 waits without counting an attempt while no orch strand is recorded. But the dobby's existing post seams turn "no strand recorded" into an ordinary error through `requirePosted`, so a watch seam built the same way counts the wait as a failed attempt and drops the line after three probes. The record does not say the seam must report no-strand apart from a failure.
  - Evidence: `internal/beamcli/dobbydeps.go:89-96`; `internal/orchcli/notify.go:73`, where `NotifyRunFrom` returns `posted` separately.
  - Fix: in D5, state that the watch's post seam reports "no orch strand recorded" apart from a failed post, from `NotifyRunFrom`'s `posted=false` with a nil error, and does not go through `requirePosted`.

**Tool calls**

1. `requirePosted` and `postNotice` in `beamcli/dobbydeps.go` and `orchcli/notify.go`: to see how an existing seam reports "no strand recorded"; the manifest lacked these bodies.
2. `dobbyWatchDirs`, plus every `OwnedElsewhere` and `HandOverMarked` use in `dobbyengine`: to check D6's pair resolution and whether any watch-adjacent step skips runs owned elsewhere; neither was in the manifest.
3. Callers of `CheckLocalValues` and of the `beamsetting` keys outside `config.go`: to find other readers and writers of the per-task key set; the manifest lacked them.
4. `boardengine/store.go` run_settings handling and `dobbyengine/reinterpret.go`'s head: to judge whether the board's or the reinterpretation's per-task key lists need `run_watch`.
5. Grep for `[dobby notice]` and `[batten notice]` prefix constants: to check whether any code routes notices by prefix, which bears on D7; not in the manifest.
