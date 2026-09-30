---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/plan/05-exit-sweep.md
    summary: Card 5 puts the exit hook and its config load in internal/loggerconfig, a site D4 does not name; D4 licenses only cmd/lyx root or a cliwire helper
  - id: F2
    severity: MEDIUM
    location: _lyx/plan/03-register-logger-config.md
    summary: No card tests the acceptance criterion that lyx config reconcile seeds logger.yaml from the template; card 3 only extends TestNames
  - id: F3
    severity: LOW
    location: _lyx/plan/03-register-logger-config.md
    summary: Card 3 ImpactSummary names only lyx config surfaces and misses that fabric clone now seeds and commits logger.yaml into every new hub
  - id: F4
    severity: LOW
    location: _lyx/plan/05-exit-sweep.md
    summary: The never-armed integration case covers an invalid logger.yaml only; the acceptance criterion names absent too, the case where a stray load would log the arming Info record
  - id: F5
    severity: LOW
    location: _lyx/plan/04-sink-arm-state.md
    summary: Card 4's NotifyExit(1) and before-any-record cases do not state the SetDurableSinkDir redirect they need under the go test gate
  - id: F6
    severity: NIT
    location: _lyx/plan/05-exit-sweep.md
    summary: isLyxWorktree's doc comment in sink.go names cmd/lyx's direct logger.NotifyExit(code) call, which card 5 replaces without rewording it
---

### [MEDIUM] Exit hook placed outside the CLI layer D4 names

**Location:** `_lyx/plan/05-exit-sweep.md` (Create list, Intent); `_lyx/plan/00-overview.md` Shared Decisions, third bullet.

**Issue:** D4 fixes the loader site: "the CLI layer (cmd/lyx root or a cliwire helper — the plan picks the site) loads the config in the order D3 fixes and passes the bounds in."
Card 5 instead creates `internal/loggerconfig/exit.go` with `NotifyExitAndSweep`, so a config module package owns the process-exit sequence: it calls `logger.NotifyExit`, queries arm state, loads, warns and sweeps.
That is a third site the decision record does not license, and the Shared Decision states the choice without a reason.
Of the two named sites, cliwire is ruled out: its doc comment (`internal/cliwire/doc.go`) fixes its production dependency set, which excludes `internal/configengine` and so any package that loads config.
`cmd/lyx` root remains, and it already carries a CLI-layer helper file of the same kind (`cmd/lyx/stencilseed.go`, run from the root's pre-run).
`cmd/lyx` tests use no `t.Parallel()`, so exit-hook tests that redirect the global sink there stay sequential and isolated.

**Fix:** Move the exit hook to a new `cmd/lyx/exitsweep.go` (`notifyExitAndSweep(code int)` plus the unexported bounds helper) with its tests in `cmd/lyx/exitsweep_test.go`.
Keep `internal/loggerconfig` to the loader, and have card 2 export a path helper so the Warn can name the file without the unexported `moduleName`.
Update the Shared Decision, the overview prose, and card 5's `logger.go` and README wording to match.

### [MEDIUM] Reconcile seeding of logger.yaml is untested

**Location:** `_lyx/plan/03-register-logger-config.md`.

**Issue:** The acceptance criterion "`lyx config reconcile` recognises `logger.yaml` and seeds it from the template" has no test in any card.
Card 3 adds `"logger"` to `TestNames`, which pins registry membership and order only.
The `configcli` tests it cites iterate `configreg.Names()` for help text and `--print` headers, not for seeding.
The one existing test that would observe seeding, `internal/fabriccli/cloneconfigcommit_integration_test.go`, is integration-tagged in a package the verify section does not run with `-tags integration`.

**Fix:** Add `internal/configsync/configsync_test.go` to card 3's targets with a test that `ReconcileAll(dir, true)` over an empty `_lyx/config/` writes `logger.yaml` whose content equals `loggerconfig.ConfigTemplate()`, and reports a `logger` result with `Applied: true`.
`internal/configsync` has no build-tagged tests, so `go test ./...` already runs it and the verify section needs no change.

### [LOW] Card 3 ImpactSummary misses fabric clone

**Location:** `_lyx/plan/03-register-logger-config.md`, `**ImpactSummary:**`.

**Issue:** `configreg.Modules()` is also iterated by `configsync.ReconcileAll` (`internal/configsync/configsync.go:80`), which `internal/fabriccli/clone.go` calls with `apply=true` and then commits every written file via `CommitAnchoredPaths`.
So the registration makes every `lyx fabric clone` seed and commit `_lyx/config/logger.yaml` on the weft primary, and every `hubforge`-built test hub carries it.
The summary names only the `lyx config` surfaces.
No test pins a module or file list that card 3 must edit: `cloneconfigcommit_integration_test.go` derives its list from `configreg.Modules()`, `hubforge/hub_test.go` reads the `board` template only, and the `configsync`/`configcli` tests find results by name or iterate `Names()`.

**Fix:** Rewrite the summary to name both reach points and the conclusion that no pinned list needs an edit beyond `TestNames`.

### [LOW] Never-armed integration case covers only an invalid config

**Location:** `_lyx/plan/05-exit-sweep.md`, third `main_integration_test.go` case.

**Issue:** The acceptance criterion reads "A process that never armed its sink, run in a lyx worktree whose `logger.yaml` is absent or invalid, writes no trace file and loads no config at exit."
The card tests the invalid file only.
The absent file is the case D3's rationale is built on: `configengine.LoadOrTemplate` logs at Info when it degrades on an absent file (`internal/configengine/config.go`, `fallbackToTemplate`), and that Info record would arm the sink.

**Fix:** Run the `lyx --help` case twice, once with no `logger.yaml` under `_lyx/config/` and once with an invalid one, asserting no trace file and no Warn in both.

### [LOW] Card 4 omits the redirect two test cases need

**Location:** `_lyx/plan/04-sink-arm-state.md`, `sink_test.go` cases.

**Issue:** Under `go test` without `LYX_TRACE=1`, `resolveSinkDirLocked` refuses the cwd-anchored path unless an override is in force, so `NotifyExit(1)` leaves `sinkOK` false and `CurrentSinkArmState()` reports `Armed: false`.
The "After `NotifyExit(1)` with no prior record, it reports `Armed: true`" case holds only after `SetDurableSinkDir(t.TempDir())`, and the "creates no file" assertion of the first case needs a redirected directory to inspect.
The card states the redirect for the `Info` case and not for these two.
`sink_test.go`'s own header convention (every case calls `SetDurableSinkDir(t.TempDir())` first) makes the intent recoverable, but the card should say it.

**Fix:** State the `SetDurableSinkDir(t.TempDir())` redirect for every case, and assert `Redirected: true` and `Dir` in the `NotifyExit(1)` case.

### [NIT] Stale NotifyExit reference in sink.go

**Location:** `_lyx/plan/05-exit-sweep.md`; the target comment is `isLyxWorktree`'s doc comment in `internal/logger/sink.go`.

**Issue:** That comment says "cmd/lyx's logger.NotifyExit(code) force-arms the sink on EVERY non-zero exit".
After card 5, `cmd/lyx` calls its exit hook, which calls `logger.NotifyExit`; the comment names a call site that no longer exists as written.

**Fix:** Have card 5 reword it to "cmd/lyx's exit hook, through logger.NotifyExit(code), force-arms …".

## Focus-directive verdicts

- Card 3 blast radius: the ImpactSummary misses fabric clone (F3), and no test pins a list card 3 must edit (F3's evidence).
- Verify coverage: not a defect.
  Card 3 needs no target in `internal/configcli` or `internal/fabriccli`, and F2's fix adds `internal/configsync`, which has no build-tagged tests.
  Every targeted package with integration-tagged tests (`internal/logger`, `cmd/lyx`) is in the `-tags integration` line.
- Exit-hook placement: outside what D4 licenses (F1).
- Redirects: card 5 states its redirect in the `exit_test.go` preamble, and every case can be written as specified; card 4 does not (F5).
- Acceptance walk: every criterion maps to a card test except reconcile seeding (F2), and the never-armed criterion's absent-file variant is missing (F4).
  "Exactly once at exit (zero and non-zero)" is carried jointly by card 5's arm-no-longer-sweeps `sink_test.go` case and its code-0-after-Info and code-1 exit cases.
