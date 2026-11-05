```yaml
lens: repo-invariants
outcome: reported
```

Task: repo-invariants lens over the amended cards 07, 08, 23 and 25, plus the prefix-only change to 26.

**Per card, each invariant checked:**

- **Card 07 (report-config)**
  - Test seams, held. The card adds no field to a shared struct. `config_test.go` has one tier tag (`tiergate.Require(t, 0)`, line 18). `testmain_test.go` takes no tier tag and uses `tmuxkit.Main`.
  - Configreg tests, held. `TestRegistration_MatchesDeclarers` scans every `ConfigTemplate` declarer, so registering `reportconfig` in `Modules` satisfies it (`registration_test.go`:28-50). `TestNames` checks order and shape only, so dropping it from the card's Edit list is correct (`configreg_test.go`:20-45).
  - Doc text, held:
    - The card lists `pattern/PATTERN-config-layers.md`'s Degrading callers set.
    - The card lists `configStrictnessDegradingSet`.
    - `configreg/doc.go` names no modules.
  - Doc text, broken (finding F1). The new package needs a line in the `docs/overview.md` package tree. Neither card 07 nor card 08 names that line in its Intent.
  - Card order, held. Nothing earlier is needed.
- **Card 08 (report-cost)**
  - Card order, held. `reportconfig#LoadConfig` comes from card 07, which is earlier.
  - Import invariants, held:
    - `reportengine#allowedImports` gains nothing, because the config import sits in `reportcli` (`internal/reportcli/cost.go`:12).
    - `reportcli` has no import allowlist (grep found none).
  - Doc text, partly broken (F1). The card's Intent names the report module line and the tree line "for the cost verb". It does not name the `+ internal/reportconfig` package in the module line or the new `internal/reportconfig/` tree line.
- **Card 23 (raddle-recipe)**
  - Doc text, held:
    - The five recipe-list `doc.go` files now sit under Edit, and each named sentence exists (`loomcli/doc.go`:25, `shedrun/doc.go`:5, `loomrecipe/doc.go`:34/48/63, `shedcli/doc.go`:12, `runprofile/doc.go`:3/12).
    - `shedrecipe/doc.go` stays under Extends. Its gate-fixer sentence (line 117) is not falsified, because the raddle gate carries no `fixer`.
  - Test seams, held. The card's reflective seam tests are unchanged by the amendment and still listed.
  - Card order, held.
- **Card 25 (pawl-raddle-handoff)**
  - Doc text, held:
    - `pawlengine/doc.go` is now under Edit, and both sentences it names exist (lines 8 and 23).
    - `hubgeom/doc.go` lists no pawl paths, so leaving it under Uses is correct.
  - Test seams and card order, held. The amendment did not change them.
- **Card 26 (report-raddle)**
  - Only `plan:` prefixes were dropped, for symbols that now exist. Card order holds: `usageFiles` and `costCommand` come from card 08, which is earlier.

**Findings**

- **F1**
  - Severity: LOW. Class: consistency. Location: `_lym/plan/07-report-config.md` Intent and `_lym/plan/08-report-cost.md`:45.
  - Summary: Card 07 creates the package `internal/reportconfig`, and the package needs its own line in the `docs/overview.md` package tree. The report module line also needs to name `internal/reportconfig`. Card 07 does not list `docs/overview.md`. Card 08 edits that file, but its Intent names the overview lines only for the cost verb, so neither card states either edit.
  - Evidence: the package tree lists every internal package (`docs/overview.md`:240/318, the `ideconfig` and `hibernateconfig` lines). Commit 807263d5 added `docs/overview.md`:319 (the `reportconfig` tree line) and :480 (`+ internal/reportconfig`) under card 08, without a plan line covering them.
  - Fix: card 08's Intent names the new `internal/reportconfig/` tree line and the module line's `+ internal/reportconfig`. Its Edit list already has `//docs/overview.md`, so no target changes.

**Tool calls**

1. grep of `configreg/doc.go`, `docs/overview.md`, the `reportcli` allowlist and `hubgeom/doc.go`: the manifest did not cover the docs that list modules, packages or pawl paths.
2. `git show 807263d5 -- docs/overview.md` and a search for tests that read the overview: to find which commit added the `reportconfig` overview lines and whether a test enforces them.
3. `shedrecipe/doc.go` lines 110–124: to check whether card 23's Extends on that file falsifies the sentence saying which entries read the gate-fixer key.
