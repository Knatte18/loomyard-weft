```yaml
lens: existing-paths
outcome: reported
```

Lens: existing-paths. I traced every path that writes, lifts or reads the state D1 to D12 introduce or change.

**Paths traced, per decision**

- D1, worker under `runproc.HubOwner`:
  - Anchor `EndDobbies` → `runproc.EndSparing(..., RolePush, RoleSync)` ends it: **survives**, because the record states it and D2 resumes. `internal/fabriccli/anchor.go:468`, `internal/fabriccli/anchordeps.go:258`
  - Tender reap: **survives**, because `orphanOwner` never treats `HubOwner` as an orphan. `internal/tenderengine/processes.go:119`
  - Tender `EndOwner` and slug `End`: **survives**, because they end only orphan owners or slug owners. `internal/tendercli/deps.go:114`, `:342`
  - Pair teardown `EndSparing`: **survives**, because its owner is a slug. `internal/pairteardown/teardown.go:690`
  - Beam stop `EndRole(RoleDobby)`: **survives**, because it ends only the dobby role. `internal/orchcli/cli.go:276`
- D2/D9, `adopt.json` and `adopt.lock` directly under `AdoptScratchDir`:
  - `ReadCheckViews` skips anything that is not a directory: **survives**. `internal/adoptcli/checkview.go:56`
  - No other walker of that directory: **survives**. Only `internal/adoptcli/check.go:140` and `checkview.go` use it.
- D4, owner exits:
  - Orch watcher `ownerTick`, which D4 names: **survives**.
  - Dobby `beamOwnedElsewhere` exit, which D4 names: **survives**. `internal/dobbyengine/loop.go:494`
  - Pass-level `ownedElsewhere` owner pause on an in-flight pass: **survives**, because D5 converts the resulting `owner` entry in place. `internal/dobbyengine/owned.go`
  - `lym beam start` `prepare` → `pauseOwnedRuns` in the window: **survives** for moved runs, through the same conversion. `internal/orchcli/start.go:395`
  - Watcher and dobby started by an older binary: **survives**, because both re-exec on a replaced binary between ticks. `internal/orchengine/watchloop.go:221`, `internal/dobbyengine/loop.go:65`
- D5, the `adopt` caller entry:
  - Dobby `planPass` `run.Held` → `passNone`, and `recheckOwnerPause` gated on `!run.Held`: **survives**, provided `Held` is widened to `adopt` as D5 says. `internal/dobbyengine/passes.go:120`, `:191`, `internal/battencli/dobbyreads.go:168`
  - Dobby `LiftOwnerPause` judges only `OwnerCaller`: **survives**. `internal/pauserecord/doc.go:24`
  - Hibernate wake `holdRuns` pauses with `OwnerCaller` and adds none when a slug entry stands: **survives**. `internal/hibernateengine/wake.go:104-113`
  - Cleanup `LiftWakeHold` lifts only `hibernate` entries of ended runs: **survives**. `internal/cleanupcli/reconcile.go:74`
  - InnerRun parent-lock release on `OwnerHeld`, push-child `OwnerEntryExempt`, cancel follow-up `OwnerPauseHeld`: **survives, behaviour as today**. Today's conversion already turns `owner` into `adopt-hold` in place (`internal/adoptcli/doc.go:111`). `internal/battenshed/innerrun_parentlock.go:36`, `internal/pauserecord/hold.go:306`, `internal/pairteardown/followup.go:58`
  - `lym orch status` watcher restart and tender `reviveOrch` act only on a recorded strand, so neither starts an orch D7 skipped: **survives**. `internal/orchcli/status.go:84`, `internal/tenderengine/resume.go:499`
- D3, dropping the settled-runs step:
  - Today's `settled` list covers the adopted beam slugs as well as the moved runs, and `resumeSlug` / `liftOwnerPauses` lifts each one's `owner` entry: **breaks** for the beam's own pair entry (F1). `internal/adoptcli/adopt.go:398-407`, `:1005-1020`
- D8, green end: an older check reading `holds.json` is stated as D5's bound: **survives**.
- D11, `archive-owner/` tags:
  - Records-tag reads for `selectRuns`, cost, raddle and timing go through `TagRefs(ArchiveTagPrefix)`; cleanupremotewarp and `remoteArchiveTags` use `refs/tags/archive/&lt;slug&gt;/`. None match `archive-owner/`: **survives**. `internal/reportengine/source.go:77`, `internal/fabricengine/cleanupremotewarp.go:239`, `internal/fabricengine/remoteleftover.go:224`
  - A respawned dobby (anchor `EnsureDobbies`) repeats the archive under the same deterministic tag name and exits: **survives**.

**Findings**

F1
- Severity: MEDIUM
- Class: design
- Location: `_lym/discussion/decision-record.md` D3 "Today's settled-runs step … is dropped" with D5's pause set
- Summary: An adopted beam's own pair `owner` pause entry is no longer lifted or converted by anything, and the record is silent about it.
- Evidence:
  - Today's settled list starts from `adopted`, the beam slugs, and `liftOwnerPauses` lifts each slug's own-scope `OwnerCaller` entry. `internal/adoptcli/adopt.go:398-407`, `:1005-1020`
  - Such an entry exists whenever a reconcile on this machine read the beam `owned_elsewhere`. `pauseOwnedRuns` pauses every `EntryOwnedElsewhere` entry, beam projects included, for example on a `lym beam start` in the prime. `internal/orchcli/start.go:395-411`
  - D5 converts `owner` entries only for moved runs (open entries with an unfinished batten record) and released tasks, so a beam project with no run record is not covered.
  - The dobby's owner lift reaches only runs it drives that carry `OwnedElsewhere` state. `internal/dobbyengine/passes.go:191`
- Effect: after adopt, the beam pair's sessions, including the orch that D7 or `beam start` brings up, stay covered by an `owner` entry. Only a `lym tender resume &lt;beam&gt;` that nothing names lifts it.
- What the record must decide: either D5 also converts that entry to `adopt` and the run report or D9 names it, or the adopt lifts it as a non-run entry. Then the record states which.

**Tool calls**

1. grep for `OwnerCaller`, `.Held`, `OwnerHeld`, `holds.json` and `AdoptDirName` across `internal` and `cmd`: the manifest lacked every reader and writer of the owner and hold state.
2. Read `pauserecord/hold.go` `OwnerEntryExempt`, `battenshed/innerrun_parentlock.go`, `battencli/cancel.go` follow-up, `pairteardown/followup.go` and `dobbyengine/passes.go` `planPass`: these are the readers call 1 surfaced, which the manifest did not hold.
3. grep for orchcli status restart, and read tender `reviveOrch`: to check whether an existing path re-starts an orch that D7 skipped.
4. Read `orchcli/start.go` `pauseOwnedRuns`, `adoptcli/adopt.go` settled list and `liftOwnerPauses`: to see which slugs today's dropped step lifted and which entries beam start writes.
