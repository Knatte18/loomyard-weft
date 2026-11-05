```yaml
lens: affected-set
outcome: reported
```

Lens: affected-set. I traced what each card in the amendment (07, 08, 23, 25) forces outside itself. The sets are almost fully covered; there is one LOW gap.

**Per change: the set found and the cards that cover it**

- **Card 07: new hub-wide module `report` in the new package `internal/reportconfig`**
  - `configreg#Modules`: covered by card 07 (Edit).
  - `TestModules_SeedOnlyAndHubWideFlags`, its `wantHubWide` literal and its doc comment, which lists the hub-wide modules: covered by card 07 (Edit).
  - `TestRegistration_MatchesDeclarers` (`internal/configreg/registration_test.go:28`) finds the new `ConfigTemplate` declarer through `Modules()` by itself. No card is needed.
  - `TestNames` (`internal/configreg/configreg_test.go:23`) checks order and shape only, not membership. Card 07 rightly dropped its edit of it.
  - `cmd/lym#configStrictnessDegradingSet` (`cmd/lym/configstrictness_test.go:64`): covered by card 07 (Edit).
  - The Degrading callers line in `pattern/PATTERN-config-layers.md` (line 63): covered by card 07 (Edit).
  - `reportengine#allowedImports` (`internal/reportengine/seam_enforcement_test.go:14`): no longer affected. `reportengine` does not import `reportconfig`; only `internal/reportcli/cost.go:12` does, and `reportcli` has no import allowlist. Card 07 rightly dropped the edit.
  - `TestFingerprint` (`configreg_test.go:323`) uses synthetic modules. Not affected.
  - In `docs/overview.md`, the package-tree line for `internal/reportconfig` (line 319) and the package list in the report module line (line 480): **no card's Intent names either.** Card 08 lists `//docs/overview.md` as an Edit target, but its Intent says only that the overview's "report module line and tree line … name the cost verb". Commit 807263d5 added both lines anyway.
  - Other docs that name config files (`docs/overview.md:499`, `contracts/orchrole/orch-role.md:266`) are specific to hibernate. Not affected.
- **Card 08: `reportconfig#LoadConfig` replaces `reportengine#LoadConfig`**
  - The only reader is `costCommand` (`internal/reportcli/cost.go:59`): covered by card 08.
  - The `reportcli` doc line (`internal/reportcli/doc.go:26`): covered by card 08 (Edit of `internal/reportcli/doc.go`).
- **Card 23: the doc files that list recipes move from Extends to Edit, and the Intent names each sentence**
  - `internal/loomcli/doc.go:25`
  - `internal/shedrun/doc.go:5`
  - `internal/loomrecipe/doc.go:34`, `:48` and `:63`
  - `internal/shedcli/doc.go:12`
  - `internal/runprofile/doc.go:3` and `:12`
  - All are covered by card 23's Edit list and its Intent.
  - `internal/shedrecipe/doc.go` stays under Extends and gains raddle entry sections. It is covered, with no recipe list to rewrite.
- **Card 25: `pawlengine/doc.go` moves to Edit**
  - Line 8 (the Beam-Up step list) and line 23 (the list of files under `.lym/beam/`): both covered by the Intent sentence card 25 added.

**Findings**

1. **LOW, consistency, card 08 (`08-report-cost`, `//docs/overview.md`).** The new package's `docs/overview.md` entries are covered only in part. Card 08's Intent names the overview lines only as gaining the cost verb. It omits two members of the set card 07's new package forces:
   - the package-tree line for `internal/reportconfig/`, at `docs/overview.md:319`;
   - the addition of `internal/reportconfig` to the report module line's package list, at `docs/overview.md:480`.

   Card 07, which creates the package, does not target the overview either. Commit 807263d5 shows that the implementer wrote both lines anyway.

   Evidence: `_lym/plan/08-report-cost.md:45`, `_lym/plan/07-report-config.md:13-17`, and `git show 807263d5 -- docs/overview.md`.

Tool calls:
1. Grepped for files naming `hibernateconfig`. The manifest did not list which docs and lists name the sibling config packages.
2. Grepped `docs/overview.md`, `configreg/doc.go` and `PATTERN-config-layers.md` for config package names, and ran `git show 807263d5 -- docs/overview.md`. I needed the overview lines the new package forces and which commit wrote them.
3. Grepped `reportcli` and `configreg` for import allowlists, and `docs`, `configcli` and `contracts` for lists of config files. I needed to confirm no other allowlist or module list reads the new module.
