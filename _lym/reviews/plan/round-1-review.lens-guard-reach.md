```yaml
lens: guard-reach
outcome: reported
```

Task: guard-reach lens. For each guard, refusal or branch the amendment's cards (07, 08, 23, 25) add, I checked whether a real entry point can reach it and whether a test does.

**Per guard: reachable or not**

- **Card 07, `reportconfig.LoadConfig` refuses `cost_min_runs` &lt; 1.** Reachable: a hub `report.yaml` holding 0 gets past `LoadOrTemplate` and the YAML decode and hits the check at `internal/reportconfig/config.go:58`. `TestLoadConfig` has a zero-value row. Held.
- **Card 08, `costCommand` refuses a bad `--since` or `--format`.** Reachable from the CLI, and `TestReportVerbs_RefuseBadFlags` gains the cost rows. Held.
- **Card 08, `ComputeCost` skips a run with no `run.json` and lists it as a finding.** Reachable, for example a run whose usage directory was written before card 6 seeded `run.json`. There is a `TestComputeCost` row. Held.
- **Card 08, cancelled or torn-down run left out; unknown-cost session gives hours only.** Both reachable, and each has a row. Held.
- **Card 25, `HandOff` error leads to Stuck/Halt.** Reachable through the seam, and a `TestBeamUp` row covers it. Held.
- **Card 25, `Notify` error after a found line leads to Halt with no marker.** Reachable, and a row covers it. Held.
- **Card 25, post reaches no orch (`posted` false, nil error) leads to Halt with no marker.** Reachable in the real path: `NotifyOrchAt` returns posted false when no orch strand is recorded (`internal/orchcli/notify.go:68`). The fake has a row for it. Held.
- **Card 25, marker present means no second mail; marker absent after an interruption means a second mail.** Both reachable, and `TestBeamUp_RerunIsIdempotent` covers both. The mail is sent only from `BeamUp.Call`; `awaitRun.notify` sends the await notices and stays outside the guard, which is intended. Held.
- **Card 25, `beamUpEntry` / `requirePawl` refuse an unset `HandOff`.** Reachable, and `TestNewProducers_RefuseAnUnsetSeam` plus the `simpleEntryCases` row cover it. Held.
- **Card 23, `raddleFileEntry` refuses an unknown key or an unset seam.** Reachable from recipe YAML, and `TestRaddleEntries` covers both. Held.
- **Card 23, `raddleWriteEntry` refuses a key other than `gates`.** Reachable: nothing upstream rejects row keys first, and `discussionWriteEntry` relies on the same `configRejectUnknown` at `internal/shedrecipe/entries_discussionwrite.go:35`. No named test reaches it, because `TestRaddleEntries` lists an unknown-key row for `raddleFileEntry` only. **Finding F-GR1.**
- **Card 23, `createBeamFromParent` falls back to the prime when `hubgeom.ParentOf` names no parent.** Reachable: a Sketch whose parent is the prime has no beam above it, which is the common case for a top-level Sketch. `TestCreateBeamFromParent` has only two rows, a run in the parent worktree and a failed run, so the prime branch is never reached. **Finding F-GR2.**
- **Card 23, `unsupportedChildRecipeRefusal`.** This guard already exists; the card only extends its list. `TestChildRecipeSupported` reaches it with an unknown recipe. Held.

**Findings**

- **F-GR1**
  - Severity: LOW. Class: consistency.
  - Location: `_lym/plan/23-raddle-recipe.md:99` and `:110`.
  - Summary: `raddleWriteEntry` rejects unknown keys other than `gates`, but no `TestRaddleEntries` row reaches that refusal; only `raddleFileEntry`'s unknown-key refusal has a row.
  - Evidence: the card's Intent at line 99 against its test list at line 110; the shared helper at `internal/shedrecipe/entries_discussionwrite.go:35`.
  - Fix: add a `raddleWriteEntry` unknown-key row to `TestRaddleEntries`.
- **F-GR2**
  - Severity: LOW. Class: consistency.
  - Location: `_lym/plan/23-raddle-recipe.md:103` and `:111`.
  - Summary: `createBeamFromParent`'s branch for "the prime when `ParentOf` names none" is reachable for any prime-parented Sketch, but no `TestCreateBeamFromParent` row reaches it.
  - Evidence: the card's Intent at line 103 against its test rows at line 111; the empty-parent path in `internal/hubgeom/parentof.go:21`.
  - Fix: add a row for a slug with no parent beam, where the child runs in the prime.

**Tool calls**

1. `internal/reportconfig/config.go:40-80` and `orchcli.NotifyOrchAt`: to confirm the refusal sits after the decode and to see when `posted` comes back false. My context had only the declaration lines.
2. `pawlengine.BeamUp.Call` and `pawlDeps`' Notify wiring: to trace where the hand-off step sits and how the posted result is dropped today. Neither was in the manifest.
3. `discussionWriteEntry`, the shedrecipe unknown-key tests and `hubgeom.ParentOf`: to check whether anything upstream rejects row keys and whether an empty parent is reachable. None of these were in the manifest.
