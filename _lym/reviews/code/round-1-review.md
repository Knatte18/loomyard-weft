---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/passes.go:heartbeat
    summary: >-
      a failed Runs read at the start heartbeat leaves the logged-run memory unprimed, so a re-exec then logs a spurious change line for every run
    fix: >-
      In `heartbeat`, derive `first` from whether the logged-run memory has been primed rather than from `lastHeartbeat.IsZero()`. Either set `d.lastHeartbeat = now` only after the `d.deps.Runs()` read succeeds (a failed start read then leaves `first` true for the next heartbeat), or keep a separate `loggedRunsPrimed` bool set where `loggedRuns` is filled. Add a row to the heartbeat test where the start `Runs` read fails and the next heartbeat primes `loggedRuns`, so the following `logRunChanges` logs no change for an unchanged run.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/configsync/configsync.go:957
    summary: >-
      a hub that pins adopt to the template value through lym config gate has it dropped by the per-name drop and reseeded from finalize on the next reconcile, so the way back to the template that the record names does not hold
    fix: >-
      In `ReconcileHubWideAt`, keep the per-name `adopt` entry of the hub `gate.yaml` out of `yamlengine.DropTemplateEqualNames` (or out of `stripBase`) whenever the file holds `gates.finalize`, so a template-equal `adopt` stays present and `SeedAdoptGate` never sees it absent. Pin it with a `TestReconcileHubWideAt` row where a hub tunes `finalize` to [0, 1] and sets `adopt` to `{tiers: [0], scope: all}`; two reconcile passes must leave `adopt` at `[0]`. If keeping the entry is judged wrong, the alternative is to correct the sentence in `gateslot/doc.go` and the `SeedAdoptGate` comment so they say the template value cannot be pinned while `finalize` is tuned.
  - id: F3
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: internal/adoptcli/adopt.go:1
    summary: >-
      the file header still says adopt starts the test suite, though it now starts the adopt check that runs the adopt gate
    fix: >-
      In the file header of `internal/adoptcli/adopt.go`, replace `starts the test suite` with `starts the adopt check`, as the `deps.go` header and the `StartSuite` comment now say.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/dobbyengine/loop.go:Deps.Log
    summary: >-
      the Deps.Log doc comment is written in terms of another named symbol, Geometry.LogPath
    fix: >-
      In the `Deps.Log` doc comment, drop `and the dobby passes Geometry.LogPath`; say only that rotate is set for the dobby's own lines while it holds its lock.
---

### [LOW] Heartbeat leaves the logged-run memory unprimed after a failed start read

**Location:** `internal/dobbyengine/passes.go`, `heartbeat`
**Issue:** `heartbeat` sets `d.lastHeartbeat = now` before it reads the runs, and takes `first` from `lastHeartbeat.IsZero()`.
When the start heartbeat's `Runs()` read fails, the function returns after writing `heartbeat: runs unreadable`, and `loggedRuns` is never filled.
The next heartbeat sees `first` false, so the memory stays empty.
After a re-exec the first `logRunChanges` then logs every listed run as first seen, which is the spurious line Decision 6 and the acceptance criterion `After a re-exec, the first tick logs no state-change line for a run whose state is unchanged` rule out.
It needs a failed `Runs` read at exactly the start, so it is rare.

### [LOW] A template-equal adopt pinned by the hub is dropped and reseeded

**Location:** `internal/configsync/configsync.go`, the `MigrateHubBase` call and the `DropTemplateEqualNames` call after it in `ReconcileHubWideAt`
**Issue:** `SeedAdoptGate` runs before the per-name drop.
A hub that tunes `finalize` and sets `adopt` to the template value (`{tiers: [0], scope: all}`) through `lym config gate` has that entry dropped as template-equal on the same pass, because the seed saw it present.
The next reconcile finds `adopt` absent with `finalize` tuned and seeds it with `finalize`'s value.
The Open risks section says a hub returns to the template's `adopt` by setting it through `lym config gate`; for the template value that does not hold, and `adopt` flips to `finalize`'s tiers one build later.

### [NIT] adopt.go file header names the old test suite

**Location:** `internal/adoptcli/adopt.go:1`
**Issue:** The header ends with `starts the test suite`.
The diff changed what the file starts and updated the `deps.go` header, the `StartSuite` comment and the `startSuite` doc, but not this line.

### [NIT] Deps.Log doc names another symbol

**Location:** `internal/dobbyengine/loop.go`, the `Deps.Log` field comment
**Issue:** The comment ends `and the dobby passes Geometry.LogPath`.
A doc comment is self-contained and is not written in terms of another named symbol.

## Rejected

- Code-correctness F3, an undecodable state record replacing the kept tidy with nil in `readKeptTidy`: Decision 1 keeps the tidy `nil included` and the record reads as absent when it does not decode; `loop_worktreegone_test.go` pins the row `an undecodable record reads as no tidy`, and a record that stays undecodable also fails every tick.
- Code-tests, no `TestMigrateConfig` row for `MigrateConfig` no longer seeding `adopt`: the row exists in the diff of `internal/gateslot/config_test.go` as `a tuned finalize with no adopt gains none, since only the hub base seed adds it`.
- Code-pattern-fit F-a, the bare `.git` in `worktreeGone`: Decision 1 prescribes the worktree's git link joined with `.git`, and the stat runs no git.
- Code-pattern-fit side note, `unix.Dup2` not building on linux/arm64: `GOOS=linux GOARCH=arm64 go vet ./internal/beamcli/` passes.
